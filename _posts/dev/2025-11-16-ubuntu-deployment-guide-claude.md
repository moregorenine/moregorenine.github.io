---
title: "Ubuntu Web Service 배포 가이드 (개선판)"
excerpt: "Oracle Cloud Ubuntu 프로덕션 배포 완벽 가이드 - Zero-downtime 배포 포함"
categories: 
  - dev
tags: 
  - dev
  - devops
  - deployment
last_modified_at: 2025-11-16T00:00:00+09:00
toc: true
toc_sticky: true
---

# Oracle Cloud Ubuntu 프로덕션 배포 가이드

## 아키텍처 개요

```
                      [Oracle Cloud - 공인 IP]
                                |
                        [OCI 보안 목록]
                      (포트: 80, 443, 22)
                                |
                      [Ubuntu VM 인스턴스]
                                |
                      [Nginx (80/443)]
                      SSL: Let's Encrypt
                                |
         +----------------------+----------------------+
         |                      |                      |
    [/worklog]             [/worklog1]            [/worklog2]
     :5173, :5174           :5175, :5176           :5177, :5178
     (Node.js)              (Node.js)              (Node.js)
         |                      |                      |
         +----------------------+----------------------+
                                |
                            [/api]
                        :8090, :8095
                      (Spring Boot JAR)
                                |
                     +----------+----------+
                     |                     |
               [PostgreSQL]           [Redis]
                 :5432                 :6379
              (Docker Network)     (Docker Network)
```

## 1단계: OCI 인스턴스 설정

### 컴퓨트 인스턴스 생성

**인스턴스 사양 (권장):**

- **Shape**: VM.Standard.E2.1.Micro (무료 티어) 또는 VM.Standard.E4.Flex
- **이미지**: Canonical Ubuntu 22.04 LTS
- **OCPU**: 1-2
- **메모리**: 8-16 GB
- **부트 볼륨**: 100 GB (여유 공간 확보)
- **VCN**: 신규 생성 또는 기존 사용
- **공인 IP**: 공인 IPv4 할당 필수

### OCI 보안 목록 규칙 (인그레스)

VCN의 보안 목록에 다음 규칙 추가:

**SSH (임시, 배포 후 특정 IP로 제한)**
- Stateless: No
- Source: 0.0.0.0/0 → 나중에 관리자 IP로 변경
- IP Protocol: TCP
- Source Port Range: All
- Destination Port Range: 22

**HTTP**
- Stateless: No
- Source: 0.0.0.0/0
- IP Protocol: TCP
- Source Port Range: All
- Destination Port Range: 80

**HTTPS**
- Stateless: No
- Source: 0.0.0.0/0
- IP Protocol: TCP
- Source Port Range: All
- Destination Port Range: 443

## 2단계: 초기 Ubuntu 설정

### 인스턴스 SSH 접속

```bash
ssh ubuntu@<OCI-공인-IP>
```

### 시스템 업데이트

```bash
sudo apt update && sudo apt upgrade -y
```

### Swap 메모리 설정 (OOM 방지)

```bash
# 2GB Swap 파일 생성
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 영구 설정
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Swappiness 조정 (메모리 사용 우선)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 확인
free -h
```

### 필수 패키지 설치

```bash
sudo apt install -y curl wget git unzip build-essential ufw fail2ban htop
```

### Node.js 22.x 설치

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# 확인
node --version  # v22.x
npm --version
```

### Java 17 설치

```bash
sudo apt install -y openjdk-17-jdk

# 확인
java -version  # openjdk version "17.x"
```

### Docker & Docker Compose 설치

```bash
# Docker 설치
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu

# 새 그룹 적용 (재로그인 대신)
newgrp docker

# Docker Compose Plugin 설치
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# Docker GPG key 추가
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Docker repository 추가
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 패키지 업데이트 및 설치
sudo apt update
sudo apt install -y docker-compose-plugin

# 확인
docker --version
docker compose version
```

## 3단계: 방화벽 설정 (Ubuntu UFW)

### UFW 기본 정책 설정

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 필수 포트 허용

```bash
# SSH (나중에 특정 IP로 제한)
sudo ufw allow 22/tcp

# HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### UFW 활성화

```bash
sudo ufw --force enable
sudo ufw status verbose
```

### 방화벽 확인

```bash
sudo ufw status numbered
```

## 4단계: Nginx 설치 및 기본 설정

### Nginx 설치

```bash
sudo apt install -y nginx

# 기본 사이트 비활성화
sudo systemctl stop nginx
sudo systemctl disable nginx  # 나중에 수동으로 활성화
```

## 5단계: PostgreSQL & Redis 설정 (Docker)

### 디렉토리 구조 생성

```bash
sudo mkdir -p /opt/moremong/{docker,backend,frontend,frontend1,frontend2,backups,versions}
sudo chown -R ubuntu:ubuntu /opt/moremong
```

### Docker Compose 파일 생성

```bash
cat > /opt/moremong/docker/docker-compose.yml <<'EOF'
services:
  postgresql:
    image: postgres:17.4
    container_name: moremong-postgres
    restart: always
    environment:
      POSTGRES_DB: moremong
      POSTGRES_USER: moremong
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "127.0.0.1:5432:5432"
    networks:
      - moremong-network
    command:
      - "postgres"
      - "-c"
      - "max_connections=200"
      - "-c"
      - "shared_buffers=256MB"
      - "-c"
      - "effective_cache_size=1GB"
      - "-c"
      - "maintenance_work_mem=64MB"
      - "-c"
      - "checkpoint_completion_target=0.9"
      - "-c"
      - "wal_buffers=16MB"
      - "-c"
      - "default_statistics_target=100"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U moremong"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7.2.3
    container_name: moremong-redis
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - "127.0.0.1:6379:6379"
    networks:
      - moremong-network
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  moremong-network:
    driver: bridge
EOF
```

### 환경 변수 파일 생성

```bash
cat > /opt/moremong/docker/.env <<'EOF'
POSTGRES_PASSWORD=CHANGE_THIS_STRONG_PASSWORD_1
REDIS_PASSWORD=CHANGE_THIS_STRONG_PASSWORD_2
EOF

# 보안 권한 설정
chmod 600 /opt/moremong/docker/.env
```

⚠️ **중요**: 실제 강력한 비밀번호로 변경하세요!

```bash
# 비밀번호 생성 예시
openssl rand -base64 32
```

### Docker 서비스 시작

```bash
cd /opt/moremong/docker
docker compose up -d

# 상태 확인
docker compose ps
docker compose logs -f
```

### PostgreSQL 초기 설정 확인

```bash
# PostgreSQL 접속 테스트
docker exec -it moremong-postgres psql -U moremong -d moremong

# psql 프롬프트에서
\l  # 데이터베이스 목록
\q  # 종료
```

## 6단계: 애플리케이션 파일 배포

### 방법 1: Git Clone

```bash
cd /opt/moremong
git clone <저장소-URL> repo
```

### 방법 2: SCP 업로드

```bash
# 로컬 머신에서 실행
scp -r ./your-app ubuntu@<OCI-공인-IP>:/opt/moremong/repo
```

### 방법 3: rsync (증분 업데이트)

```bash
# 로컬 머신에서 실행
rsync -avz --progress ./your-app/ ubuntu@<OCI-공인-IP>:/opt/moremong/repo/
```

## 7단계: 백엔드 빌드 & 배포

### 프로덕션 환경 파일 생성

```bash
cat > /opt/moremong/repo/restapi/.env.production <<'EOF'
# Jasypt 암호화 마스터 키
JASYPT_ENCRYPTOR_PASSWORD=YOUR_MASTER_ENCRYPTION_KEY

# Spring Profile
SPRING_PROFILES_ACTIVE=prod

# 데이터베이스
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/moremong
SPRING_DATASOURCE_USERNAME=moremong
SPRING_DATASOURCE_PASSWORD=CHANGE_THIS_STRONG_PASSWORD_1

# Redis
SPRING_DATA_REDIS_HOST=localhost
SPRING_DATA_REDIS_PORT=6379
SPRING_DATA_REDIS_PASSWORD=CHANGE_THIS_STRONG_PASSWORD_2

# OAuth (암호화된 값으로 대체)
# KAKAO_CLIENT_ID=ENC(...)
# KAKAO_CLIENT_SECRET=ENC(...)
# NAVER_CLIENT_ID=ENC(...)
# NAVER_CLIENT_SECRET=ENC(...)
# GOOGLE_CLIENT_ID=ENC(...)
# GOOGLE_CLIENT_SECRET=ENC(...)
EOF

chmod 600 /opt/moremong/repo/restapi/.env.production
```

### JAR 빌드

```bash
cd /opt/moremong/repo/restapi

# 권한 설정 (필요시)
chmod +x gradlew

# 프로덕션 빌드
./gradlew -Pprod clean bootJar

# 빌드 확인
ls -lh build/libs/*.jar
```

### 첫 배포

```bash
# JAR 파일 복사
cp build/libs/*.jar /opt/moremong/backend/moremong-restapi.jar

# 환경 파일 복사
cp .env.production /opt/moremong/backend/.env

# 권한 설정
chmod 600 /opt/moremong/backend/.env
```

### Systemd 서비스 생성

**백엔드 인스턴스 1 (포트 8090)**

```bash
sudo tee /etc/systemd/system/moremong-api-1.service > /dev/null <<'EOF'
[Unit]
Description=Moremong Backend API Instance 1
After=network.target docker.service
Wants=docker.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/moremong/backend
EnvironmentFile=/opt/moremong/backend/.env
Environment="SERVER_PORT=8090"

ExecStart=/usr/bin/java \
  -server \
  -Xms1g \
  -Xmx2g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+UseStringDeduplication \
  -XX:+OptimizeStringConcat \
  -Djava.security.egd=file:/dev/./urandom \
  -Dspring.profiles.active=prod \
  -jar /opt/moremong/backend/moremong-restapi.jar

Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-api-1

# 로그 제한
LogRateLimitIntervalSec=30s
LogRateLimitBurst=1000

[Install]
WantedBy=multi-user.target
EOF
```

**백엔드 인스턴스 2 (포트 8095)**

```bash
sudo tee /etc/systemd/system/moremong-api-2.service > /dev/null <<'EOF'
[Unit]
Description=Moremong Backend API Instance 2
After=network.target docker.service
Wants=docker.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/moremong/backend
EnvironmentFile=/opt/moremong/backend/.env
Environment="SERVER_PORT=8095"

ExecStart=/usr/bin/java \
  -server \
  -Xms1g \
  -Xmx2g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+UseStringDeduplication \
  -XX:+OptimizeStringConcat \
  -Djava.security.egd=file:/dev/./urandom \
  -Dspring.profiles.active=prod \
  -jar /opt/moremong/backend/moremong-restapi.jar

Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-api-2

# 로그 제한
LogRateLimitIntervalSec=30s
LogRateLimitBurst=1000

[Install]
WantedBy=multi-user.target
EOF
```

## 8단계: 프론트엔드 빌드 & 배포

### 프로덕션 환경 파일 생성

```bash
cd /opt/moremong/repo/moremong-front

cat > .env.production <<'EOF'
PUBLIC_API_URL=https://moremong.com
EOF

chmod 644 .env.production
```

### SvelteKit 설정 확인

`svelte.config.js` 파일에서 adapter와 paths 설정 확인:

```javascript
import adapter from '@sveltejs/adapter-node';

/** @type {import('@sveltejs/kit').Config} */
const config = {
  kit: {
    adapter: adapter(),
    paths: {
      base: process.env.BASE_PATH || ''
    }
  }
};

export default config;
```

### 빌드 (worklog)

```bash
# BASE_PATH를 환경변수로 설정하고 빌드
BASE_PATH=/worklog npm run build

# 빌드 파일 복사
cp -r build /opt/moremong/frontend/
cp .env.production /opt/moremong/frontend/.env
cp package.json package-lock.json /opt/moremong/frontend/

# 프로덕션 의존성 설치
cd /opt/moremong/frontend
npm ci --omit=dev
```

### Systemd 서비스 생성

**프론트엔드 인스턴스 1 (포트 5173)**

```bash
sudo tee /etc/systemd/system/moremong-front-1.service > /dev/null <<'EOF'
[Unit]
Description=Moremong Frontend /worklog Instance 1
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/moremong/frontend
EnvironmentFile=/opt/moremong/frontend/.env
Environment="PORT=5173"
Environment="HOST=127.0.0.1"
Environment="ORIGIN=https://moremong.com"

ExecStart=/usr/bin/node build

Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-front-1

# 로그 제한
LogRateLimitIntervalSec=30s
LogRateLimitBurst=1000

[Install]
WantedBy=multi-user.target
EOF
```

**프론트엔드 인스턴스 2 (포트 5174)**

```bash
sudo tee /etc/systemd/system/moremong-front-2.service > /dev/null <<'EOF'
[Unit]
Description=Moremong Frontend /worklog Instance 2
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/moremong/frontend
EnvironmentFile=/opt/moremong/frontend/.env
Environment="PORT=5174"
Environment="HOST=127.0.0.1"
Environment="ORIGIN=https://moremong.com"

ExecStart=/usr/bin/node build

Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-front-2

# 로그 제한
LogRateLimitIntervalSec=30s
LogRateLimitBurst=1000

[Install]
WantedBy=multi-user.target
EOF
```

### 추가 프론트엔드 인스턴스 (필요시)

**frontend1 (worklog1) - 포트 5175, 5176**

```bash
cd /opt/moremong/repo/moremong-front1
BASE_PATH=/worklog1 npm run build
cp -r build /opt/moremong/frontend1/
cp .env.production /opt/moremong/frontend1/.env
cp package.json package-lock.json /opt/moremong/frontend1/
cd /opt/moremong/frontend1
npm ci --omit=dev
```

유사하게 systemd 서비스 생성 (포트 5175, 5176 사용)

**frontend2 (worklog2) - 포트 5177, 5178**

```bash
cd /opt/moremong/repo/moremong-front2
BASE_PATH=/worklog2 npm run build
cp -r build /opt/moremong/frontend2/
cp .env.production /opt/moremong/frontend2/.env
cp package.json package-lock.json /opt/moremong/frontend2/
cd /opt/moremong/frontend2
npm ci --omit=dev
```

유사하게 systemd 서비스 생성 (포트 5177, 5178 사용)

## 9단계: SSL 인증서 발급 (Let's Encrypt)

### Certbot 설치

```bash
sudo apt install -y certbot python3-certbot-nginx
```

### SSL 인증서 발급

⚠️ **중요**: 도메인 DNS가 OCI 공인 IP를 가리켜야 합니다!

```bash
# 실제 도메인으로 변경
sudo certbot certonly --standalone \
  -d moremong.com \
  -d www.moremong.com \
  --non-interactive \
  --agree-tos \
  -m your-email@example.com \
  --preferred-challenges http

# 인증서 위치 확인
sudo ls -l /etc/letsencrypt/live/moremong.com/
```

### 자동 갱신 설정

```bash
# Certbot 타이머 활성화 (자동 갱신)
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# 타이머 상태 확인
sudo systemctl status certbot.timer

# 갱신 테스트 (dry-run)
sudo certbot renew --dry-run
```

### 갱신 후 Nginx 리로드 설정

```bash
sudo tee /etc/letsencrypt/renewal-hooks/deploy/nginx-reload.sh > /dev/null <<'EOF'
#!/bin/bash
systemctl reload nginx
EOF

sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/nginx-reload.sh
```

## 10단계: Nginx 설정

### Nginx 메인 설정 파일

```bash
sudo tee /etc/nginx/nginx.conf > /dev/null <<'EOF'
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log warn;

events {
    worker_connections 2048;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'rt=$request_time uct="$upstream_connect_time" '
                    'uht="$upstream_header_time" urt="$upstream_response_time"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 20M;
    server_tokens off;  # 버전 정보 숨김

    # Gzip 압축
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript
               application/json application/javascript application/xml+rss
               application/rss+xml font/truetype font/opentype
               application/vnd.ms-fontobject image/svg+xml;
    gzip_disable "msie6";

    # 보안 헤더
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=general:10m rate=30r/s;
    limit_req_zone $binary_remote_addr zone=api:10m rate=50r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

    # 업스트림 백엔드 (로드 밸런싱)
    upstream backend_api {
        least_conn;
        server 127.0.0.1:8090 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:8095 max_fails=3 fail_timeout=30s;
        keepalive 32;
    }

    # 업스트림 프론트엔드
    upstream frontend_worklog {
        least_conn;
        server 127.0.0.1:5173 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:5174 max_fails=3 fail_timeout=30s;
        keepalive 16;
    }

    upstream frontend_worklog1 {
        least_conn;
        server 127.0.0.1:5175 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:5176 max_fails=3 fail_timeout=30s;
        keepalive 16;
    }

    upstream frontend_worklog2 {
        least_conn;
        server 127.0.0.1:5177 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:5178 max_fails=3 fail_timeout=30s;
        keepalive 16;
    }

    # HTTP -> HTTPS 리다이렉트
    server {
        listen 80;
        listen [::]:80;
        server_name moremong.com www.moremong.com;

        # Let's Encrypt ACME 챌린지
        location /.well-known/acme-challenge/ {
            root /var/www/html;
        }

        # 나머지는 HTTPS로 리다이렉트
        location / {
            return 301 https://$host$request_uri;
        }
    }

    # HTTPS 메인 서버
    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name moremong.com www.moremong.com;

        # SSL 인증서
        ssl_certificate /etc/letsencrypt/live/moremong.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/moremong.com/privkey.pem;
        ssl_session_timeout 1d;
        ssl_session_cache shared:MozSSL:10m;
        ssl_session_tickets off;

        # 최신 SSL 프로토콜 및 암호화
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
        ssl_prefer_server_ciphers off;

        # HSTS (HTTP Strict Transport Security)
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

        # 백엔드 API
        location /api {
            limit_req zone=api burst=50 nodelay;

            proxy_pass http://backend_api;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header X-Forwarded-Port $server_port;

            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
            proxy_buffering off;

            # CORS 헤더 (필요시)
            # add_header Access-Control-Allow-Origin "https://moremong.com" always;
        }

        # OAuth2 로그인 엔드포인트
        location ~ ^/(oauth2|login) {
            limit_req zone=login burst=10 nodelay;

            proxy_pass http://backend_api;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header X-Forwarded-Port $server_port;

            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }

        # Management 엔드포인트 (로컬만 접근)
        location /management {
            allow 127.0.0.1;
            deny all;

            proxy_pass http://backend_api;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header Host $host;
        }

        # 프론트엔드 /worklog
        location /worklog {
            limit_req zone=general burst=50 nodelay;

            proxy_pass http://frontend_worklog;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 프론트엔드 /worklog1
        location /worklog1 {
            limit_req zone=general burst=50 nodelay;

            proxy_pass http://frontend_worklog1;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 프론트엔드 /worklog2
        location /worklog2 {
            limit_req zone=general burst=50 nodelay;

            proxy_pass http://frontend_worklog2;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 루트 경로 리다이렉트
        location = / {
            return 301 https://$host/worklog;
        }

        # 헬스 체크 엔드포인트
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }

        # 파비콘 무시 (404 방지)
        location = /favicon.ico {
            log_not_found off;
            access_log off;
        }
    }
}
EOF
```

### Nginx 설정 테스트

```bash
sudo nginx -t
```

**아직 시작하지 마세요!** 모든 서비스가 준비된 후 시작합니다.

## 11단계: 모든 서비스 시작

### Systemd 데몬 리로드

```bash
sudo systemctl daemon-reload
```

### 백엔드 서비스 시작

```bash
# 활성화 (부팅 시 자동 시작)
sudo systemctl enable moremong-api-1 moremong-api-2

# 서비스 시작
sudo systemctl start moremong-api-1
sleep 10  # 첫 인스턴스가 완전히 시작되길 기다림

sudo systemctl start moremong-api-2

# 상태 확인
sudo systemctl status moremong-api-1
sudo systemctl status moremong-api-2

# 로그 확인
sudo journalctl -u moremong-api-1 -n 50 --no-pager
sudo journalctl -u moremong-api-2 -n 50 --no-pager

# 헬스 체크
curl http://localhost:8090/management/health
curl http://localhost:8095/management/health
```

### 프론트엔드 서비스 시작

```bash
# 활성화 및 시작
sudo systemctl enable moremong-front-1 moremong-front-2
sudo systemctl start moremong-front-1 moremong-front-2

# 상태 확인
sudo systemctl status moremong-front-1
sudo systemctl status moremong-front-2

# 헬스 체크
curl -I http://localhost:5173/worklog
curl -I http://localhost:5174/worklog
```

### Nginx 시작

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx

# 설정 리로드 (설정 변경 시)
sudo nginx -s reload
```

### 전체 서비스 확인

```bash
# 모든 서비스 상태
sudo systemctl status moremong-api-1 moremong-api-2 moremong-front-1 moremong-front-2 nginx

# Docker 컨테이너 확인
docker compose -f /opt/moremong/docker/docker-compose.yml ps

# 포트 리스닝 확인
sudo netstat -tuln | grep LISTEN
```

## 12단계: OAuth 제공자 설정

각 OAuth 제공자 콘솔에서 리다이렉트 URI를 업데이트하세요.

### Google Cloud Console

1. [Google Cloud Console](https://console.cloud.google.com/) 접속
2. API 및 서비스 > 사용자 인증 정보
3. OAuth 2.0 클라이언트 ID 선택
4. **승인된 리다이렉트 URI** 추가:
   - `https://moremong.com/login/oauth2/code/google`

### Kakao Developers

1. [Kakao Developers](https://developers.kakao.com/) 접속
2. 내 애플리케이션 > 앱 설정 > 플랫폼
3. **Redirect URI** 추가:
   - `https://moremong.com/login/oauth2/code/kakao`

### Naver Developers

1. [Naver Developers](https://developers.naver.com/) 접속
2. 내 애플리케이션 > API 설정
3. **Callback URL** 추가:
   - `https://moremong.com/login/oauth2/code/naver`

### 백엔드 application.yml 확인

Spring Boot의 `application.yml` 또는 `application-prod.yml`에 다음 설정이 있는지 확인:

```yaml
application:
  oauth2:
    allowed-redirect-uris:
      - https://moremong.com/worklog/oauth/callback
      - https://moremong.com/worklog1/oauth/callback
      - https://moremong.com/worklog2/oauth/callback

spring:
  security:
    oauth2:
      client:
        registration:
          google:
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
          kakao:
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
          naver:
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
```

## 13단계: 보안 강화

### 1. Fail2ban 설정

```bash
sudo tee /etc/fail2ban/jail.local > /dev/null <<'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
destemail = your-email@example.com
sendername = Fail2Ban-Moremong

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 3

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
EOF

# Fail2ban 활성화
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# 상태 확인
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### 2. SSH 강화

```bash
# SSH 설정 백업
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# SSH 보안 강화 적용
sudo tee -a /etc/ssh/sshd_config > /dev/null <<'EOF'

# === 보안 강화 설정 ===
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
PrintMotd no
AcceptEnv LANG LC_*

# 연결 타임아웃
ClientAliveInterval 300
ClientAliveCountMax 2

# 인증 시도 제한
MaxAuthTries 3
MaxSessions 5
LoginGraceTime 60

# 허용 사용자 (선택적)
# AllowUsers ubuntu
EOF

# SSH 재시작
sudo systemctl restart sshd

# SSH 상태 확인
sudo systemctl status sshd
```

⚠️ **주의**: SSH 재시작 전에 다른 터미널로 접속 테스트를 먼저 하세요!

### 3. 자동 보안 업데이트

```bash
# Unattended Upgrades 설치
sudo apt install -y unattended-upgrades

# 설정
sudo tee /etc/apt/apt.conf.d/50unattended-upgrades > /dev/null <<'EOF'
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";

Unattended-Upgrade::Mail "your-email@example.com";
Unattended-Upgrade::MailReport "on-change";
EOF

# 자동 업데이트 활성화
sudo dpkg-reconfigure -plow unattended-upgrades
```

### 4. 파일 권한 강화

```bash
# 애플리케이션 디렉토리
sudo chown -R ubuntu:ubuntu /opt/moremong
sudo chmod -R 755 /opt/moremong

# 민감한 환경 파일
sudo chmod 600 /opt/moremong/backend/.env
sudo chmod 600 /opt/moremong/frontend/.env
sudo chmod 600 /opt/moremong/docker/.env

# Nginx 설정
sudo chown -R root:root /etc/nginx
sudo chmod 644 /etc/nginx/nginx.conf

# SSL 인증서
sudo chmod 644 /etc/letsencrypt/live/moremong.com/fullchain.pem
sudo chmod 600 /etc/letsencrypt/live/moremong.com/privkey.pem
```

### 5. 감사 로깅 (Auditd)

```bash
# Auditd 설치
sudo apt install -y auditd

# 감사 규칙 추가
sudo tee /etc/audit/rules.d/moremong.rules > /dev/null <<'EOF'
# 시스템 파일 변경 감시
-w /etc/passwd -p wa -k passwd_changes
-w /etc/group -p wa -k group_changes
-w /etc/sudoers -p wa -k sudoers_changes
-w /etc/ssh/sshd_config -p wa -k sshd_config_changes

# 애플리케이션 디렉토리 변경 감시
-w /opt/moremong -p wa -k moremong_changes

# Nginx 설정 변경 감시
-w /etc/nginx -p wa -k nginx_changes

# systemd 서비스 변경 감시
-w /etc/systemd/system -p wa -k systemd_changes
EOF

# Auditd 재시작
sudo systemctl restart auditd
sudo systemctl enable auditd

# 감사 로그 확인
sudo ausearch -k passwd_changes
```

### 6. 로그 로테이션

```bash
# Nginx 로그 로테이션
sudo tee /etc/logrotate.d/moremong-nginx > /dev/null <<'EOF'
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 `cat /var/run/nginx.pid`
        fi
    endscript
}
EOF

# 애플리케이션 로그 로테이션 (journald가 관리하지만 추가 설정 가능)
sudo tee /etc/logrotate.d/moremong-app > /dev/null <<'EOF'
/var/log/moremong/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 ubuntu ubuntu
}
EOF

# 로그 로테이션 테스트
sudo logrotate -d /etc/logrotate.d/moremong-nginx
```

### 7. 불필요한 서비스 비활성화

```bash
# 사용하지 않는 서비스 확인
systemctl list-unit-files --type=service --state=enabled

# 예시: Bluetooth, CUPS 등 비활성화
sudo systemctl disable bluetooth.service 2>/dev/null || true
sudo systemctl disable cups.service 2>/dev/null || true

# 불필요한 패키지 제거
sudo apt autoremove -y
```

## 14단계: 모니터링 설정

### 1. Node Exporter 설치 (Prometheus용)

```bash
# 최신 버전 다운로드
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-1.7.0.linux-amd64*

# Systemd 서비스 생성
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<'EOF'
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/local/bin/node_exporter \
  --web.listen-address=127.0.0.1:9100

Restart=always

[Install]
WantedBy=multi-user.target
EOF

# 서비스 시작
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# 확인
curl http://localhost:9100/metrics
```

### 2. 로그 모니터링 도구

```bash
# lnav 설치 (로그 뷰어)
sudo apt install -y lnav

# 사용 예시
sudo lnav /var/log/nginx/*.log
sudo journalctl -u moremong-api-1 -u moremong-api-2 -f
```

### 3. 헬스 체크 스크립트

```bash
cat > /opt/moremong/health-check.sh <<'EOF'
#!/bin/bash

echo "================================================"
echo "    Moremong 시스템 헬스 체크"
echo "    타임스탬프: $(date '+%Y-%m-%d %H:%M:%S')"
echo "================================================"
echo ""

# 서비스 상태
echo "=== 서비스 상태 ==="
printf "%-25s %s\n" "백엔드 API 1:" "$(systemctl is-active moremong-api-1)"
printf "%-25s %s\n" "백엔드 API 2:" "$(systemctl is-active moremong-api-2)"
printf "%-25s %s\n" "프론트엔드 1:" "$(systemctl is-active moremong-front-1)"
printf "%-25s %s\n" "프론트엔드 2:" "$(systemctl is-active moremong-front-2)"
printf "%-25s %s\n" "Nginx:" "$(systemctl is-active nginx)"
printf "%-25s %s\n" "PostgreSQL:" "$(docker ps --filter name=moremong-postgres --format '{{.Status}}' | head -c 20)"
printf "%-25s %s\n" "Redis:" "$(docker ps --filter name=moremong-redis --format '{{.Status}}' | head -c 20)"
echo ""

# 포트 리스닝
echo "=== 포트 리스닝 상태 ==="
netstat -tuln | grep -E ':(80|443|5173|5174|8090|8095|5432|6379) ' | awk '{print $4}' | sort -u
echo ""

# 디스크 사용량
echo "=== 디스크 사용량 ==="
df -h / | tail -n 1
echo ""

# 메모리 사용량
echo "=== 메모리 사용량 ==="
free -h | grep -E '^Mem|^Swap'
echo ""

# CPU 로드
echo "=== CPU 로드 ==="
uptime
echo ""

# Docker 컨테이너 상태
echo "=== Docker 컨테이너 ==="
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
echo ""

# 최근 에러 로그 (있을 경우)
echo "=== 최근 에러 로그 (1시간) ==="
ERROR_COUNT=$(sudo journalctl --since "1 hour ago" -p err --no-pager | wc -l)
if [ $ERROR_COUNT -gt 0 ]; then
    echo "⚠️  에러 로그 ${ERROR_COUNT}개 발견"
    sudo journalctl --since "1 hour ago" -p err --no-pager | tail -n 10
else
    echo "✓ 에러 없음"
fi
echo ""

echo "================================================"
EOF

chmod +x /opt/moremong/health-check.sh

# 실행 테스트
/opt/moremong/health-check.sh
```

### 4. 크론잡으로 정기 체크

```bash
# 매시간 헬스 체크 수행 및 로그 저장
(crontab -l 2>/dev/null; echo "0 * * * * /opt/moremong/health-check.sh >> /var/log/moremong-health.log 2>&1") | crontab -

# 크론 확인
crontab -l
```

## 15단계: 백업 전략

### 1. 데이터베이스 백업 스크립트

```bash
cat > /opt/moremong/backup-db.sh <<'EOF'
#!/bin/bash

set -e  # 에러 시 중단

BACKUP_DIR="/opt/moremong/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
POSTGRES_CONTAINER="moremong-postgres"
REDIS_CONTAINER="moremong-redis"

mkdir -p $BACKUP_DIR

echo "================================================"
echo "  데이터베이스 백업 시작: $TIMESTAMP"
echo "================================================"

# PostgreSQL 백업
echo "PostgreSQL 백업 중..."
docker exec $POSTGRES_CONTAINER pg_dump -U moremong moremong | \
  gzip > $BACKUP_DIR/postgres_$TIMESTAMP.sql.gz

if [ $? -eq 0 ]; then
    echo "✓ PostgreSQL 백업 완료: postgres_$TIMESTAMP.sql.gz"
else
    echo "✗ PostgreSQL 백업 실패!"
    exit 1
fi

# Redis 백업
echo "Redis 백업 중..."
docker exec $REDIS_CONTAINER redis-cli --rdb /data/dump.rdb SAVE
docker cp $REDIS_CONTAINER:/data/dump.rdb $BACKUP_DIR/redis_$TIMESTAMP.rdb

if [ $? -eq 0 ]; then
    echo "✓ Redis 백업 완료: redis_$TIMESTAMP.rdb"
else
    echo "✗ Redis 백업 실패!"
    exit 1
fi

# 7일 이전 백업 삭제
echo "오래된 백업 정리 중..."
find $BACKUP_DIR -name "postgres_*.sql.gz" -mtime +7 -delete
find $BACKUP_DIR -name "redis_*.rdb" -mtime +7 -delete

# 백업 파일 크기 확인
echo ""
echo "=== 백업 파일 ==="
ls -lh $BACKUP_DIR | tail -n 5

echo ""
echo "백업 완료: $TIMESTAMP"
echo "================================================"
EOF

chmod +x /opt/moremong/backup-db.sh

# 백업 테스트
/opt/moremong/backup-db.sh
```

### 2. 자동 백업 크론잡

```bash
# 매일 새벽 2시에 백업 수행
(crontab -l 2>/dev/null; echo "0 2 * * * /opt/moremong/backup-db.sh >> /var/log/moremong-backup.log 2>&1") | crontab -

# 확인
crontab -l
```

### 3. 백업 복원 스크립트

```bash
cat > /opt/moremong/restore-db.sh <<'EOF'
#!/bin/bash

set -e

BACKUP_DIR="/opt/moremong/backups"

if [ $# -ne 1 ]; then
    echo "사용법: $0 <백업파일_타임스탬프>"
    echo "예시: $0 20250116_020000"
    echo ""
    echo "사용 가능한 백업:"
    ls -lh $BACKUP_DIR | grep postgres
    exit 1
fi

TIMESTAMP=$1
POSTGRES_BACKUP="$BACKUP_DIR/postgres_${TIMESTAMP}.sql.gz"
REDIS_BACKUP="$BACKUP_DIR/redis_${TIMESTAMP}.rdb"

if [ ! -f "$POSTGRES_BACKUP" ]; then
    echo "✗ PostgreSQL 백업 파일을 찾을 수 없습니다: $POSTGRES_BACKUP"
    exit 1
fi

echo "================================================"
echo "  데이터베이스 복원 시작: $TIMESTAMP"
echo "================================================"
echo ""
read -p "⚠️  기존 데이터가 삭제됩니다. 계속하시겠습니까? (yes/no): " CONFIRM

if [ "$CONFIRM" != "yes" ]; then
    echo "복원 취소됨."
    exit 0
fi

# PostgreSQL 복원
echo "PostgreSQL 복원 중..."
docker exec moremong-postgres psql -U moremong -d postgres -c "DROP DATABASE IF EXISTS moremong;"
docker exec moremong-postgres psql -U moremong -d postgres -c "CREATE DATABASE moremong;"
gunzip < $POSTGRES_BACKUP | docker exec -i moremong-postgres psql -U moremong -d moremong

echo "✓ PostgreSQL 복원 완료"

# Redis 복원 (선택적)
if [ -f "$REDIS_BACKUP" ]; then
    echo "Redis 복원 중..."
    docker cp $REDIS_BACKUP moremong-redis:/data/dump.rdb
    docker restart moremong-redis
    echo "✓ Redis 복원 완료"
fi

echo ""
echo "================================================"
echo "  복원 완료: $TIMESTAMP"
echo "================================================"
EOF

chmod +x /opt/moremong/restore-db.sh
```

### 4. 애플리케이션 버전 백업

```bash
# 버전 관리 디렉토리
mkdir -p /opt/moremong/versions

# 배포 시 백업 스크립트
cat > /opt/moremong/backup-version.sh <<'EOF'
#!/bin/bash

VERSION=$1
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

if [ -z "$VERSION" ]; then
    echo "사용법: $0 <버전명>"
    echo "예시: $0 v1.0.0"
    exit 1
fi

BACKUP_DIR="/opt/moremong/versions/${VERSION}_${TIMESTAMP}"
mkdir -p $BACKUP_DIR

# JAR 파일 백업
cp /opt/moremong/backend/moremong-restapi.jar $BACKUP_DIR/

# 프론트엔드 빌드 백업
cp -r /opt/moremong/frontend/build $BACKUP_DIR/frontend-build

echo "✓ 버전 백업 완료: $BACKUP_DIR"
EOF

chmod +x /opt/moremong/backup-version.sh
```

## 16단계: Zero-Downtime 배포 전략

### 개요

무중단 배포는 사용자에게 서비스 중단 없이 새 버전을 배포하는 전략입니다. Nginx 로드 밸런서를 활용하여 롤링 업데이트를 수행합니다.

### 배포 프로세스

```
1. 새 버전 빌드
2. 인스턴스 1 중지 (Nginx가 트래픽을 인스턴스 2로 라우팅)
3. 인스턴스 1 업데이트 및 시작
4. 헬스 체크 통과 확인
5. 인스턴스 2 중지 (Nginx가 트래픽을 인스턴스 1로 라우팅)
6. 인스턴스 2 업데이트 및 시작
7. 헬스 체크 통과 확인
8. 배포 완료
```

### Zero-Downtime 배포 스크립트 (백엔드)

```bash
cat > /opt/moremong/deploy-backend.sh <<'EOF'
#!/bin/bash

set -e

VERSION=${1:-"unknown"}
JAR_PATH=${2:-"/opt/moremong/repo/restapi/build/libs/moremong-restapi.jar"}

if [ ! -f "$JAR_PATH" ]; then
    echo "✗ JAR 파일을 찾을 수 없습니다: $JAR_PATH"
    exit 1
fi

echo "================================================"
echo "  백엔드 Zero-Downtime 배포 시작"
echo "  버전: $VERSION"
echo "  JAR: $JAR_PATH"
echo "================================================"
echo ""

# 현재 버전 백업
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/opt/moremong/versions/${VERSION}_${TIMESTAMP}"
mkdir -p $BACKUP_DIR
cp /opt/moremong/backend/moremong-restapi.jar $BACKUP_DIR/moremong-restapi.jar.backup
echo "✓ 현재 버전 백업 완료: $BACKUP_DIR"
echo ""

# 새 JAR 복사
cp $JAR_PATH /opt/moremong/backend/moremong-restapi.jar.new
echo "✓ 새 JAR 파일 복사 완료"
echo ""

# 인스턴스 1 업데이트
echo "=== 인스턴스 1 업데이트 ==="
echo "인스턴스 1 중지 중..."
sudo systemctl stop moremong-api-1

echo "JAR 교체 중..."
mv /opt/moremong/backend/moremong-restapi.jar /opt/moremong/backend/moremong-restapi.jar.old
mv /opt/moremong/backend/moremong-restapi.jar.new /opt/moremong/backend/moremong-restapi.jar

echo "인스턴스 1 시작 중..."
sudo systemctl start moremong-api-1

# 헬스 체크 대기
echo "헬스 체크 대기 중..."
for i in {1..30}; do
    if curl -s -f http://localhost:8090/management/health > /dev/null 2>&1; then
        echo "✓ 인스턴스 1 헬스 체크 통과 (${i}초)"
        break
    fi
    echo -n "."
    sleep 1
done
echo ""

if ! curl -s -f http://localhost:8090/management/health > /dev/null 2>&1; then
    echo "✗ 인스턴스 1 헬스 체크 실패! 롤백 중..."
    sudo systemctl stop moremong-api-1
    mv /opt/moremong/backend/moremong-restapi.jar.old /opt/moremong/backend/moremong-restapi.jar
    sudo systemctl start moremong-api-1
    exit 1
fi

sleep 5
echo ""

# 인스턴스 2 업데이트
echo "=== 인스턴스 2 업데이트 ==="
echo "인스턴스 2 중지 중..."
sudo systemctl stop moremong-api-2

echo "인스턴스 2 시작 중..."
sudo systemctl start moremong-api-2

# 헬스 체크 대기
echo "헬스 체크 대기 중..."
for i in {1..30}; do
    if curl -s -f http://localhost:8095/management/health > /dev/null 2>&1; then
        echo "✓ 인스턴스 2 헬스 체크 통과 (${i}초)"
        break
    fi
    echo -n "."
    sleep 1
done
echo ""

if ! curl -s -f http://localhost:8095/management/health > /dev/null 2>&1; then
    echo "✗ 인스턴스 2 헬스 체크 실패!"
    exit 1
fi

# 정리
rm -f /opt/moremong/backend/moremong-restapi.jar.old

echo ""
echo "================================================"
echo "  ✓ 백엔드 배포 완료"
echo "  버전: $VERSION"
echo "  타임스탬프: $TIMESTAMP"
echo "================================================"

# 배포 후 확인
echo ""
echo "=== 배포 후 확인 ==="
sudo systemctl status moremong-api-1 --no-pager | grep Active
sudo systemctl status moremong-api-2 --no-pager | grep Active
curl -s http://localhost:8090/management/health | head -n 1
curl -s http://localhost:8095/management/health | head -n 1
EOF

chmod +x /opt/moremong/deploy-backend.sh
```

### Zero-Downtime 배포 스크립트 (프론트엔드)

```bash
cat > /opt/moremong/deploy-frontend.sh <<'EOF'
#!/bin/bash

set -e

VERSION=${1:-"unknown"}
BUILD_PATH=${2:-"/opt/moremong/repo/moremong-front/build"}
FRONTEND_NAME=${3:-"frontend"}  # frontend, frontend1, frontend2
SERVICE_PREFIX=${4:-"moremong-front"}  # moremong-front, moremong-front1, moremong-front2

if [ ! -d "$BUILD_PATH" ]; then
    echo "✗ 빌드 디렉토리를 찾을 수 없습니다: $BUILD_PATH"
    exit 1
fi

DEPLOY_DIR="/opt/moremong/$FRONTEND_NAME"

echo "================================================"
echo "  프론트엔드 Zero-Downtime 배포 시작"
echo "  버전: $VERSION"
echo "  프론트엔드: $FRONTEND_NAME"
echo "================================================"
echo ""

# 현재 버전 백업
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/opt/moremong/versions/${FRONTEND_NAME}_${VERSION}_${TIMESTAMP}"
mkdir -p $BACKUP_DIR
cp -r $DEPLOY_DIR/build $BACKUP_DIR/build.backup
echo "✓ 현재 버전 백업 완료: $BACKUP_DIR"
echo ""

# 새 빌드 복사
cp -r $BUILD_PATH $DEPLOY_DIR/build.new
echo "✓ 새 빌드 복사 완료"
echo ""

# 인스턴스 1 업데이트
echo "=== 인스턴스 1 업데이트 ==="
echo "인스턴스 1 중지 중..."
sudo systemctl stop ${SERVICE_PREFIX}-1

echo "빌드 교체 중..."
mv $DEPLOY_DIR/build $DEPLOY_DIR/build.old
mv $DEPLOY_DIR/build.new $DEPLOY_DIR/build

echo "인스턴스 1 시작 중..."
sudo systemctl start ${SERVICE_PREFIX}-1

# 헬스 체크 대기
echo "헬스 체크 대기 중..."
PORT1=$(systemctl show ${SERVICE_PREFIX}-1 -p Environment | grep -oP 'PORT=\K[0-9]+')
for i in {1..30}; do
    if curl -s -f http://localhost:${PORT1}/ > /dev/null 2>&1; then
        echo "✓ 인스턴스 1 헬스 체크 통과 (${i}초)"
        break
    fi
    echo -n "."
    sleep 1
done
echo ""

if ! curl -s -f http://localhost:${PORT1}/ > /dev/null 2>&1; then
    echo "✗ 인스턴스 1 헬스 체크 실패! 롤백 중..."
    sudo systemctl stop ${SERVICE_PREFIX}-1
    rm -rf $DEPLOY_DIR/build
    mv $DEPLOY_DIR/build.old $DEPLOY_DIR/build
    sudo systemctl start ${SERVICE_PREFIX}-1
    exit 1
fi

sleep 5
echo ""

# 인스턴스 2 업데이트
echo "=== 인스턴스 2 업데이트 ==="
echo "인스턴스 2 중지 중..."
sudo systemctl stop ${SERVICE_PREFIX}-2

echo "인스턴스 2 시작 중..."
sudo systemctl start ${SERVICE_PREFIX}-2

# 헬스 체크 대기
PORT2=$(systemctl show ${SERVICE_PREFIX}-2 -p Environment | grep -oP 'PORT=\K[0-9]+')
echo "헬스 체크 대기 중..."
for i in {1..30}; do
    if curl -s -f http://localhost:${PORT2}/ > /dev/null 2>&1; then
        echo "✓ 인스턴스 2 헬스 체크 통과 (${i}초)"
        break
    fi
    echo -n "."
    sleep 1
done
echo ""

if ! curl -s -f http://localhost:${PORT2}/ > /dev/null 2>&1; then
    echo "✗ 인스턴스 2 헬스 체크 실패!"
    exit 1
fi

# 정리
rm -rf $DEPLOY_DIR/build.old

echo ""
echo "================================================"
echo "  ✓ 프론트엔드 배포 완료"
echo "  버전: $VERSION"
echo "  타임스탬프: $TIMESTAMP"
echo "================================================"

# 배포 후 확인
echo ""
echo "=== 배포 후 확인 ==="
sudo systemctl status ${SERVICE_PREFIX}-1 --no-pager | grep Active
sudo systemctl status ${SERVICE_PREFIX}-2 --no-pager | grep Active
curl -I http://localhost:${PORT1}/ 2>/dev/null | head -n 1
curl -I http://localhost:${PORT2}/ 2>/dev/null | head -n 1
EOF

chmod +x /opt/moremong/deploy-frontend.sh
```

### 전체 배포 스크립트 (백엔드 + 프론트엔드)

```bash
cat > /opt/moremong/deploy-all.sh <<'EOF'
#!/bin/bash

set -e

VERSION=${1:-"v1.0.0"}

echo "================================================"
echo "  전체 애플리케이션 배포"
echo "  버전: $VERSION"
echo "================================================"
echo ""

# 1. 백엔드 빌드
echo "=== 1. 백엔드 빌드 ==="
cd /opt/moremong/repo/restapi
./gradlew -Pprod clean bootJar
echo "✓ 백엔드 빌드 완료"
echo ""

# 2. 프론트엔드 빌드
echo "=== 2. 프론트엔드 빌드 ==="
cd /opt/moremong/repo/moremong-front
BASE_PATH=/worklog npm run build
echo "✓ 프론트엔드 빌드 완료"
echo ""

# 3. 데이터베이스 백업
echo "=== 3. 데이터베이스 백업 ==="
/opt/moremong/backup-db.sh
echo ""

# 4. 백엔드 배포
echo "=== 4. 백엔드 배포 ==="
/opt/moremong/deploy-backend.sh $VERSION /opt/moremong/repo/restapi/build/libs/*.jar
echo ""

# 5. 프론트엔드 배포
echo "=== 5. 프론트엔드 배포 ==="
/opt/moremong/deploy-frontend.sh $VERSION /opt/moremong/repo/moremong-front/build frontend moremong-front
echo ""

# 6. 배포 확인
echo "=== 6. 배포 확인 ==="
echo "백엔드 헬스 체크:"
curl -s https://moremong.com/api/management/health | head -n 1

echo ""
echo "프론트엔드 확인:"
curl -I https://moremong.com/worklog 2>/dev/null | head -n 1

echo ""
echo "================================================"
echo "  ✓ 전체 배포 완료: $VERSION"
echo "================================================"
EOF

chmod +x /opt/moremong/deploy-all.sh
```

### 배포 사용 예시

```bash
# 백엔드만 배포
/opt/moremong/deploy-backend.sh v1.0.1 /path/to/new.jar

# 프론트엔드만 배포
/opt/moremong/deploy-frontend.sh v1.0.1 /path/to/build frontend moremong-front

# 전체 배포
/opt/moremong/deploy-all.sh v1.0.1
```

### 롤백 프로세스

```bash
cat > /opt/moremong/rollback.sh <<'EOF'
#!/bin/bash

set -e

if [ $# -ne 1 ]; then
    echo "사용법: $0 <버전_타임스탬프>"
    echo ""
    echo "사용 가능한 버전:"
    ls -lt /opt/moremong/versions/ | head -n 10
    exit 1
fi

VERSION_DIR="/opt/moremong/versions/$1"

if [ ! -d "$VERSION_DIR" ]; then
    echo "✗ 버전 디렉토리를 찾을 수 없습니다: $VERSION_DIR"
    exit 1
fi

echo "================================================"
echo "  롤백 시작: $1"
echo "================================================"
echo ""

read -p "⚠️  이전 버전으로 롤백하시겠습니까? (yes/no): " CONFIRM
if [ "$CONFIRM" != "yes" ]; then
    echo "롤백 취소됨."
    exit 0
fi

# 백엔드 롤백
if [ -f "$VERSION_DIR/moremong-restapi.jar.backup" ]; then
    echo "백엔드 롤백 중..."
    sudo systemctl stop moremong-api-1 moremong-api-2
    cp $VERSION_DIR/moremong-restapi.jar.backup /opt/moremong/backend/moremong-restapi.jar
    sudo systemctl start moremong-api-1
    sleep 10
    sudo systemctl start moremong-api-2
    echo "✓ 백엔드 롤백 완료"
fi

# 프론트엔드 롤백
if [ -d "$VERSION_DIR/frontend-build.backup" ]; then
    echo "프론트엔드 롤백 중..."
    sudo systemctl stop moremong-front-1 moremong-front-2
    rm -rf /opt/moremong/frontend/build
    cp -r $VERSION_DIR/frontend-build.backup /opt/moremong/frontend/build
    sudo systemctl start moremong-front-1
    sleep 5
    sudo systemctl start moremong-front-2
    echo "✓ 프론트엔드 롤백 완료"
fi

echo ""
echo "================================================"
echo "  ✓ 롤백 완료"
echo "================================================"
EOF

chmod +x /opt/moremong/rollback.sh
```

## 17단계: 배포 체크리스트

배포 전후 이 체크리스트를 실행하세요.

### 배포 전 체크리스트

```bash
cat > /opt/moremong/pre-deployment-checklist.sh <<'EOF'
#!/bin/bash

echo "================================================"
echo "  배포 전 체크리스트"
echo "================================================"
echo ""

PASS=0
FAIL=0

check() {
    if eval "$2"; then
        echo "✓ $1"
        ((PASS++))
    else
        echo "✗ $1"
        ((FAIL++))
    fi
}

# 1. 서비스 상태
echo "=== 서비스 상태 ==="
check "백엔드 API 1 실행 중" "systemctl is-active moremong-api-1 > /dev/null 2>&1"
check "백엔드 API 2 실행 중" "systemctl is-active moremong-api-2 > /dev/null 2>&1"
check "프론트엔드 1 실행 중" "systemctl is-active moremong-front-1 > /dev/null 2>&1"
check "프론트엔드 2 실행 중" "systemctl is-active moremong-front-2 > /dev/null 2>&1"
check "Nginx 실행 중" "systemctl is-active nginx > /dev/null 2>&1"
echo ""

# 2. Docker 컨테이너
echo "=== Docker 컨테이너 ==="
check "PostgreSQL 실행 중" "docker ps | grep moremong-postgres > /dev/null"
check "Redis 실행 중" "docker ps | grep moremong-redis > /dev/null"
echo ""

# 3. 디스크 공간
echo "=== 디스크 공간 ==="
DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')
check "디스크 사용량 < 80% (현재: ${DISK_USAGE}%)" "[ $DISK_USAGE -lt 80 ]"
echo ""

# 4. 백업 존재
echo "=== 백업 ==="
check "최근 24시간 내 백업 존재" "find /opt/moremong/backups -name 'postgres_*.sql.gz' -mtime -1 | grep -q ."
echo ""

# 5. SSL 인증서
echo "=== SSL 인증서 ==="
CERT_EXPIRY=$(sudo openssl x509 -in /etc/letsencrypt/live/moremong.com/fullchain.pem -noout -enddate | cut -d= -f2)
DAYS_LEFT=$(( ( $(date -d "$CERT_EXPIRY" +%s) - $(date +%s) ) / 86400 ))
check "SSL 인증서 만료일 > 30일 (${DAYS_LEFT}일 남음)" "[ $DAYS_LEFT -gt 30 ]"
echo ""

# 6. 네트워크
echo "=== 네트워크 ==="
check "HTTPS 접근 가능" "curl -s -f -I https://moremong.com/health > /dev/null 2>&1"
echo ""

# 결과
echo "================================================"
echo "  통과: $PASS / 실패: $FAIL"
if [ $FAIL -eq 0 ]; then
    echo "  상태: ✓ 배포 준비 완료"
    exit 0
else
    echo "  상태: ✗ 문제 해결 필요"
    exit 1
fi
echo "================================================"
EOF

chmod +x /opt/moremong/pre-deployment-checklist.sh
```

### 배포 후 체크리스트

```bash
cat > /opt/moremong/post-deployment-checklist.sh <<'EOF'
#!/bin/bash

echo "================================================"
echo "  배포 후 체크리스트"
echo "================================================"
echo ""

PASS=0
FAIL=0

check() {
    if eval "$2"; then
        echo "✓ $1"
        ((PASS++))
    else
        echo "✗ $1"
        ((FAIL++))
    fi
}

# 1. 헬스 체크
echo "=== 헬스 체크 ==="
check "백엔드 API 1 헬스" "curl -s -f http://localhost:8090/management/health > /dev/null 2>&1"
check "백엔드 API 2 헬스" "curl -s -f http://localhost:8095/management/health > /dev/null 2>&1"
check "프론트엔드 1 접근" "curl -s -f http://localhost:5173/worklog > /dev/null 2>&1"
check "프론트엔드 2 접근" "curl -s -f http://localhost:5174/worklog > /dev/null 2>&1"
echo ""

# 2. 공개 엔드포인트
echo "=== 공개 엔드포인트 ==="
check "HTTPS 메인 페이지" "curl -s -f https://moremong.com/worklog > /dev/null 2>&1"
check "API 엔드포인트" "curl -s https://moremong.com/api/management/health | grep -q UP"
echo ""

# 3. 에러 로그
echo "=== 에러 로그 (최근 10분) ==="
ERROR_COUNT=$(sudo journalctl --since "10 minutes ago" -p err --no-pager | wc -l)
check "에러 로그 < 10개 (현재: ${ERROR_COUNT}개)" "[ $ERROR_COUNT -lt 10 ]"
echo ""

# 4. 응답 시간
echo "=== 응답 시간 ==="
RESPONSE_TIME=$(curl -o /dev/null -s -w '%{time_total}' https://moremong.com/worklog)
RESPONSE_MS=$(echo "$RESPONSE_TIME * 1000" | bc | cut -d. -f1)
check "페이지 응답 시간 < 2초 (현재: ${RESPONSE_MS}ms)" "[ $RESPONSE_MS -lt 2000 ]"
echo ""

# 5. OAuth 로그인 테스트 (수동)
echo "=== OAuth 테스트 (수동 확인 필요) ==="
echo "  ⚠️  브라우저에서 다음 테스트를 수행하세요:"
echo "  1. https://moremong.com/worklog 접속"
echo "  2. 로그인 버튼 클릭"
echo "  3. OAuth 제공자 선택 및 로그인"
echo "  4. 리다이렉트 정상 작동 확인"
echo ""

# 결과
echo "================================================"
echo "  통과: $PASS / 실패: $FAIL"
if [ $FAIL -eq 0 ]; then
    echo "  상태: ✓ 배포 성공"
    exit 0
else
    echo "  상태: ✗ 문제 발생 - 조사 필요"
    exit 1
fi
echo "================================================"
EOF

chmod +x /opt/moremong/post-deployment-checklist.sh
```

## 18단계: 문제 해결 가이드

### 일반적인 문제와 해결 방법

#### 1. 서비스가 시작되지 않을 때

```bash
# 서비스 상태 확인
sudo systemctl status moremong-api-1
sudo journalctl -u moremong-api-1 -n 100 --no-pager

# 포트 충돌 확인
sudo netstat -tuln | grep 8090

# 수동 실행 (디버깅)
cd /opt/moremong/backend
java -jar moremong-restapi.jar
```

#### 2. Nginx 502 Bad Gateway

```bash
# 백엔드 서비스 확인
curl http://localhost:8090/management/health
curl http://localhost:8095/management/health

# Nginx 에러 로그
sudo tail -f /var/log/nginx/error.log

# 업스트림 연결 테스트
sudo nginx -t
```

#### 3. SSL 인증서 문제

```bash
# 인증서 확인
sudo certbot certificates

# 수동 갱신
sudo certbot renew --dry-run
sudo certbot renew

# Nginx 리로드
sudo nginx -s reload
```

#### 4. 데이터베이스 연결 실패

```bash
# PostgreSQL 컨테이너 확인
docker ps | grep postgres
docker logs moremong-postgres

# 데이터베이스 접속 테스트
docker exec -it moremong-postgres psql -U moremong -d moremong

# 연결 문자열 확인
cat /opt/moremong/backend/.env | grep DATASOURCE
```

#### 5. 디스크 공간 부족

```bash
# 디스크 사용량 확인
df -h
du -sh /opt/moremong/* | sort -h

# Docker 정리
docker system prune -a

# 오래된 로그 정리
sudo journalctl --vacuum-time=7d

# 오래된 백업 삭제
find /opt/moremong/backups -mtime +30 -delete
```

### 빠른 롤백 절차

```bash
# 1. 모든 서비스 중지
sudo systemctl stop moremong-api-1 moremong-api-2
sudo systemctl stop moremong-front-1 moremong-front-2

# 2. 백업 버전으로 복원
ls -lt /opt/moremong/versions/  # 버전 확인
/opt/moremong/rollback.sh <버전_타임스탬프>

# 3. 데이터베이스 복원 (필요시)
ls -lt /opt/moremong/backups/  # 백업 확인
/opt/moremong/restore-db.sh <타임스탬프>

# 4. 서비스 재시작
sudo systemctl start moremong-api-1 moremong-api-2
sudo systemctl start moremong-front-1 moremong-front-2

# 5. 확인
/opt/moremong/post-deployment-checklist.sh
```

## 19단계: 성능 최적화

### 1. JVM 튜닝 (프로덕션 환경)

백엔드 systemd 서비스 파일의 JVM 옵션을 최적화했습니다 (이미 7단계에 포함):

```
-server
-Xms1g
-Xmx2g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:+UseStringDeduplication
-XX:+OptimizeStringConcat
```

### 2. PostgreSQL 최적화

Docker Compose 파일에 이미 PostgreSQL 튜닝이 포함되어 있습니다 (5단계):

```yaml
command:
  - "postgres"
  - "-c"
  - "max_connections=200"
  - "-c"
  - "shared_buffers=256MB"
  # ... 기타 최적화 설정
```

### 3. Nginx 캐싱 (정적 파일)

```bash
sudo tee -a /etc/nginx/nginx.conf > /dev/null <<'EOF'

# 정적 파일 캐싱
location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
EOF

sudo nginx -t && sudo nginx -s reload
```

### 4. 커널 파라미터 최적화

```bash
sudo tee -a /etc/sysctl.conf > /dev/null <<'EOF'

# 네트워크 최적화
net.core.somaxconn = 1024
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15

# 파일 디스크립터 증가
fs.file-max = 100000
EOF

sudo sysctl -p
```

### 5. Systemd 제한 증가

```bash
# 백엔드 서비스 파일에 추가
sudo systemctl edit moremong-api-1

# 다음 내용 추가:
[Service]
LimitNOFILE=65536
LimitNPROC=4096

sudo systemctl daemon-reload
sudo systemctl restart moremong-api-1 moremong-api-2
```

## 최종 점검

### 전체 시스템 확인 스크립트

```bash
# 헬스 체크 실행
/opt/moremong/health-check.sh

# 배포 전 체크리스트
/opt/moremong/pre-deployment-checklist.sh

# 외부 접근 테스트
curl -I https://moremong.com/worklog
curl -s https://moremong.com/api/management/health

# 서비스 로그 확인 (에러 없는지)
sudo journalctl -u moremong-api-1 --since "1 hour ago" -p err --no-pager
sudo journalctl -u moremong-front-1 --since "1 hour ago" -p err --no-pager

# SSL 인증서 확인
sudo certbot certificates

# 방화벽 상태
sudo ufw status verbose

# OCI 보안 목록 확인 (웹 콘솔)
```

## 유지보수 명령어 요약

```bash
# === 서비스 관리 ===
sudo systemctl status moremong-api-1 moremong-api-2 moremong-front-1 moremong-front-2
sudo systemctl restart moremong-api-1
sudo journalctl -u moremong-api-1 -f

# === Docker 관리 ===
cd /opt/moremong/docker
docker compose ps
docker compose logs -f
docker compose restart postgresql

# === Nginx 관리 ===
sudo nginx -t
sudo nginx -s reload
sudo systemctl restart nginx
sudo tail -f /var/log/nginx/access.log

# === 백업 ===
/opt/moremong/backup-db.sh
ls -lh /opt/moremong/backups/

# === 배포 ===
/opt/moremong/deploy-all.sh v1.0.2

# === 모니터링 ===
/opt/moremong/health-check.sh
htop
docker stats
```

## 추가 리소스

### 공식 문서
- [Nginx 공식 문서](https://nginx.org/en/docs/)
- [Spring Boot 배포 가이드](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment.html)
- [SvelteKit 배포](https://kit.svelte.dev/docs/adapter-node)
- [PostgreSQL 튜닝](https://www.postgresql.org/docs/current/runtime-config.html)
- [Let's Encrypt](https://letsencrypt.org/docs/)

### 보안 체크리스트
- OCI 보안 목록 최소 권한 원칙
- SSH 키 기반 인증 사용
- 방화벽 규칙 정기 검토
- SSL/TLS 최신 버전 유지
- 정기 보안 업데이트
- 강력한 비밀번호 정책
- 로그 모니터링 및 감사

---

**작성일**: 2025-11-16  
**버전**: 2.0  
**작성자**: DevOps Team
