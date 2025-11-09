---
title: "Ubunt Web Service 설치"
excerpt: "Ubunt Web Service 설치"
categories: 
  - dev
tags: 
  - dev
last_modified_at: 2023-11-09T00:00:00+09:00
toc: true
toc_sticky: true
---

# Oracle Cloud Ubuntu 배포 가이드

## 아키텍처 개요

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
                     |                    |
               [PostgreSQL]          [Redis]
                 :5432                :6379
              (Docker/Native)      (Docker/Native)

## 1단계: OCI 인스턴스 설정

### 컴퓨트 인스턴스 생성

인스턴스 사양 (권장):

- Shape: VM.Standard.E2.1.Micro (무료 티어) 또는 VM.Standard.E4.Flex
- 이미지: Canonical Ubuntu 22.04
- OCPU: 1-2
- 메모리: 8-16 GB
- 부트 볼륨: 50-100 GB
- VCN: 신규 생성 또는 기존 사용
- 공인 IP: 공인 IPv4 할당

### OCI 보안 목록 규칙 (인그레스)

VCN의 보안 목록에 다음 규칙 추가:

- SSH (임시, 나중에 자신의 IP로 제한)
  - Stateless: No
  - Source: 0.0.0.0/0
  - IP Protocol: TCP
  - Source Port Range: All
  - Destination Port Range: 22

- HTTP
  - Stateless: No
  - Source: 0.0.0.0/0
  - IP Protocol: TCP
  - Source Port Range: All
  - Destination Port Range: 80
- HTTPS
  - Stateless: No
  - Source: 0.0.0.0/0
  - IP Protocol: TCP
  - Source Port Range: All
  - Destination Port Range: 443

## 2단계: 초기 Ubuntu 설정

인스턴스에 SSH 접속

```
ssh ubuntu@<OCI-공인-IP>
```

시스템 업데이트

```
sudo apt update && sudo apt upgrade -y
```

필수 패키지 설치

```
sudo apt install -y curl wget git unzip build-essential ufw fail2ban
```

Node.js 22.x 설치

```
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

Java 17 설치

```
sudo apt install -y openjdk-17-jdk
```

설치 확인

```
node --version  # v22.x 이어야 함
java -version   # 17.x 이어야 함
```

Docker & Docker Compose 설치 (PostgreSQL, Redis용)

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker
```

Docker Compose v2 설치

```
# 기존 패키지 업데이트
sudo apt update

# 필요 패키지 설치
sudo apt install -y ca-certificates curl gnupg lsb-release

# Docker GPG key 추가
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Docker repository 추가
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 패키지 업데이트
sudo apt update

# Docker Compose Plugin 설치
sudo apt install docker-compose-plugin
```

Docker 확인

```
docker --version
docker compose version
```

## 3단계: 방화벽 설정 (Ubuntu UFW)

UFW 방화벽 설정

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

SSH 허용 (보안 강화를 위해 22를 사용자 정의 포트로 변경 가능)

```
sudo ufw allow 22/tcp
```

HTTP/HTTPS 허용

```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

UFW 활성화

```
sudo ufw --force enable
```

상태 확인

```
sudo ufw status verbose
```

## 4단계: Nginx 설치

Nginx 설치

```
sudo apt install -y nginx
```

기본 사이트 중지

```
sudo systemctl stop nginx
```

## 5단계: PostgreSQL & Redis 설정 (Docker)

Docker Compose 파일 생성

디렉토리 생성

```
sudo mkdir -p /opt/moremong/docker
```

docker-compose.yml 생성

```
sudo tee /opt/moremong/docker/docker-compose.yml > /dev/null <<'EOF'
version: '3.8'
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
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
EOF
```

.env 파일 생성

```
sudo tee /opt/moremong/docker/.env > /dev/null <<'EOF'
POSTGRES_PASSWORD=강력한_비밀번호_1로_변경
REDIS_PASSWORD=강력한_비밀번호_2로_변경
EOF
```

보안 권한 설정

```
sudo chmod 600 /opt/moremong/docker/.env
```

서비스 시작

```
cd /opt/moremong/docker
sudo docker compose up -d
```

서비스 확인

```
sudo docker compose ps
```

Compose 프로젝트 전체 중지

```
sudo docker compose stop
```

Compose 프로젝트 전체 삭제

```
sudo docker compose down
```

상태 확인

```
sudo docker ps # 실행 중 컨테이너 확인
sudo docker compose ps # Compose 프로젝트 상태 확인
```

## 6단계: 애플리케이션 파일 배포

애플리케이션 디렉토리 생성

```
sudo mkdir -p /opt/moremong/{restapi,backend,frontend,frontend1,frontend2}
sudo chown -R ubuntu:ubuntu /opt/moremong
```

리포지토리 클론 (또는 파일 업로드)

```
cd /opt/moremong/repo
git clone <저장소-URL>
```

또는 scp/rsync로 빌드된 파일 업로드

## 7단계: 백엔드 빌드 & 배포

```
cd /opt/moremong/repo/restapi
```

프로덕션 환경 파일 생성

```
tee .env.production > /dev/null <<'EOF'
JASYPT_ENCRYPTOR_PASSWORD=여기에_마스터_비밀번호_입력
SPRING_PROFILES_ACTIVE=prod
SERVER_PORT=8090

#데이터베이스
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/moremong
SPRING_DATASOURCE_USERNAME=moremong
SPRING_DATASOURCE_PASSWORD=강력한_비밀번호_1로_변경

# Redis
SPRING_DATA_REDIS_HOST=localhost
SPRING_DATA_REDIS_PORT=6379
SPRING_DATA_REDIS_PASSWORD=강력한_비밀번호_2로_변경

# OAuth (암호화된 값 추가)
# KAKAO_CLIENT_ID=ENC(...)
# KAKAO_CLIENT_SECRET=ENC(...)
# NAVER_CLIENT_ID=ENC(...)
# NAVER_CLIENT_SECRET=ENC(...)
EOF
```

프로덕션 JAR 빌드

```
# 권한 오류 발생시
chmod +x gradlew
# build
./gradlew -Pprod clean bootJar
```

배포 디렉토리로 JAR 복사

```
cp build/libs/\*.jar /opt/moremong/backend/moremong-restapi.jar
```

환경 파일 복사

```
cp .env.production /opt/moremong/backend/.env
```

백엔드 인스턴스 1용 systemd 서비스 생성

```
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
 -Xms512m \
 -Xmx1024m \
 -Djava.security.egd=file:/dev/./urandom \
 -jar /opt/moremong/backend/moremong-restapi.jar
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-api-1

[Install]
WantedBy=multi-user.target
EOF
```

백엔드 인스턴스 2용 systemd 서비스 생성

```
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
 -Xms512m \
 -Xmx1024m \
 -Djava.security.egd=file:/dev/./urandom \
 -jar /opt/moremong/backend/moremong-restapi.jar
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-api-2

[Install]
WantedBy=multi-user.target
EOF
```

## 8단계: 프론트엔드 빌드 & 배포

```
cd /opt/moremong/repo/moremong-front
```

프로덕션 .env 생성

```
tee .env.production > /dev/null <<'EOF'
PUBLIC_API_URL=https://moremong.com
BASE_PATH=/worklog
EOF
```

BASE_PATH=/worklog로 빌드

```
BASE_PATH=/worklog npm run build
```

배포 디렉토리로 복사

```
cp -r build /opt/moremong/frontend/
cp .env.production /opt/moremong/frontend/.env
cp package.json package-lock.json /opt/moremong/frontend/
```

프로덕션 의존성 설치

```
cd /opt/moremong/frontend
npm ci --omit=dev
```

프론트엔드 인스턴스용 systemd 서비스 생성

프론트엔드 인스턴스 1 (worklog)

```
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
ExecStart=/usr/bin/node build
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-front-1

[Install]
WantedBy=multi-user.target
EOF
```

프론트엔드 인스턴스 2 (worklog)

```
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
ExecStart=/usr/bin/node build
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=moremong-front-2

[Install]
WantedBy=multi-user.target
EOF
```

frontend1과 frontend2 반복 (필요시)

BASE_PATH=/worklog1로 frontend1 빌드

```
cd /opt/moremong/repo/moremong-front1
BASE_PATH=/worklog1 npm run build
cp -r build /opt/moremong/frontend1/
```

... 포트 5175, 5176으로 유사한 systemd 서비스 생성

BASE_PATH=/worklog2로 frontend2 빌드

```
cd /opt/moremong/repo/moremong-front2
BASE_PATH=/worklog2 npm run build
cp -r build /opt/moremong/frontend2/
```

... 포트 5177, 5178으로 유사한 systemd 서비스 생성

## 9단계: Nginx 설정

SSL용 Certbot 설치

Certbot 설치

```
sudo apt install -y certbot python3-certbot-nginx
```

SSL 인증서 발급 (moremong.com을 실제 도메인으로 변경)

```
sudo certbot certonly --nginx -d moremong.com -d www.moremong.com --non-interactive --agree-tos -m your-email@example.com
```

인증서 위치:

```
/etc/letsencrypt/live/moremong.com/fullchain.pem
/etc/letsencrypt/live/moremong.com/privkey.pem
```

Nginx 설정 생성

```
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
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 20M;

    # Gzip 압축
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript
               application/json application/javascript application/xml+rss
               application/rss+xml font/truetype font/opentype
               application/vnd.ms-fontobject image/svg+xml;

    # 보안 헤더
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Rate limiting (요청 제한)
    limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=api:10m rate=20r/s;

    # 업스트림 백엔드
    upstream backend_api {
        least_conn;
        server 127.0.0.1:8090 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:8095 max_fails=3 fail_timeout=30s;
        keepalive 32;
    }
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

    # HTTP에서 HTTPS로 리다이렉트
    server {
        listen 80;
        listen [::]:80;
        server_name moremong.com www.moremong.com;
        location /.well-known/acme-challenge/ {
            root /var/www/html;
        }
        location / {
            return 301 https://$host$request_uri;
        }
    }

    # HTTPS 서버
    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name moremong.com www.moremong.com;

        # SSL 설정
        ssl_certificate /etc/letsencrypt/live/moremong.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/moremong.com/privkey.pem;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_session_tickets off;

        # 최신 SSL 설정
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-CHACHA20-POLY1305;
        ssl_prefer_server_ciphers off;

        # HSTS
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

        # 백엔드 API
        location /api {
            limit_req zone=api burst=20 nodelay;
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
        }

        # OAuth2 엔드포인트 (백엔드)
        location ~ ^/(oauth2|login) {
            limit_req zone=api burst=10 nodelay;
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

        # Management 엔드포인트 (접근 제한)
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
            limit_req zone=general burst=20 nodelay;
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
            limit_req zone=general burst=20 nodelay;
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
            limit_req zone=general burst=20 nodelay;
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

        # 루트 리다이렉트
        location = / {
            return 301 https://$host/worklog;
        }

        # 헬스 체크 엔드포인트
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
EOF
```

Nginx 설정 테스트

```
sudo nginx -t
```

아직 시작하지 말 것 - 서비스 대기

## 10단계: 모든 서비스 시작

systemd 리로드

```
sudo systemctl daemon-reload
```

백엔드 인스턴스 시작

```
sudo systemctl enable moremong-api-1 moremong-api-2
sudo systemctl start moremong-api-1 moremong-api-2
```

백엔드 상태 확인

```
sudo systemctl status moremong-api-1
sudo systemctl status moremong-api-2
```

프론트엔드 인스턴스 시작

```
sudo systemctl enable moremong-front-1 moremong-front-2
sudo systemctl start moremong-front-1 moremong-front-2
```

프론트엔드 상태 확인

```
sudo systemctl status moremong-front-1
sudo systemctl status moremong-front-2
```

Nginx 시작

```
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

로그 확인

```
sudo journalctl -u moremong-api-1 -f
sudo journalctl -u moremong-front-1 -f
```

## 11단계: 보안 강화

### 1. Fail2ban 설정

fail2ban 설치 및 설정

```
sudo tee /etc/fail2ban/jail.local > /dev/null <<'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
destemail = your-email@example.com
sendername = Fail2Ban

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
EOF
```

```
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status
```

### 2. SSH 강화

SSH 설정 백업

```
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

보안 강화 적용

```
sudo tee -a /etc/ssh/sshd_config > /dev/null <<'EOF'

#보안 강화
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
PrintMotd no
AcceptEnv LANG LC\_\*
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
MaxSessions 5
EOF

```

SSH 재시작

```
sudo systemctl restart sshd
```

### 3. 자동 보안 업데이트

```
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

자동 업데이트 활성화

```
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
EOF
```

### 4. 사용하지 않는 서비스 비활성화

사용하지 않는 서비스 목록 확인 및 비활성화

```
sudo systemctl disable bluetooth.service
sudo systemctl disable cups.service
```

불필요한 패키지 제거

```
sudo apt autoremove -y
```

### 5. 파일 권한 설정

애플리케이션 파일 보안

```
sudo chown -R ubuntu:ubuntu /opt/moremong
sudo chmod -R 755 /opt/moremong
sudo chmod 600 /opt/moremong/backend/.env
sudo chmod 600 /opt/moremong/frontend/.env
sudo chmod 600 /opt/moremong/docker/.env
```

Nginx 보안

```
sudo chown -R root:root /etc/nginx
sudo chmod 644 /etc/nginx/nginx.conf
```

### 6. 감사 로깅 활성화

```
sudo apt install -y auditd
sudo systemctl enable auditd
sudo systemctl start auditd
```

감사 규칙 추가

```
sudo tee -a /etc/audit/rules.d/audit.rules > /dev/null <<'EOF'
-w /etc/passwd -p wa -k passwd_changes
-w /etc/group -p wa -k group_changes
-w /etc/sudoers -p wa -k sudoers_changes
-w /etc/ssh/sshd_config -p wa -k sshd_config_changes
-w /opt/moremong -p wa -k moremong_changes
EOF
```

```
sudo systemctl restart auditd
```

### 7. 로그 로테이션 설정

```
sudo tee /etc/logrotate.d/moremong > /dev/null <<'EOF'
/var/log/nginx/\*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
    [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
EOF
```

## 12단계: 모니터링 설정

### 1. Node Exporter 설치 (Prometheus용)

```
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-1.7.0.linux-amd64\*
```

systemd 서비스 생성

```
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<'EOF'
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/local/bin/node_exporter \
 --web.listen-address=127.0.0.1:9100

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### 2. 로그 모니터링 설정

로그 뷰어 lnav 설치

```
sudo apt install -y lnav
```

모든 로그 확인

```
sudo lnav /var/log/nginx/\*.log
sudo journalctl -u moremong-api-1 -u moremong-front-1 -f
```

## 13단계: 백업 전략

데이터베이스 백업 스크립트

```
sudo tee /opt/moremong/backup-db.sh > /dev/null <<'EOF'
#!/bin/bash
BACKUP*DIR="/opt/moremong/backups"
TIMESTAMP=$(date +%Y%m%d*%H%M%S)
POSTGRES_CONTAINER="moremong-postgres"

mkdir -p $BACKUP_DIR

# PostgreSQL 백업
docker exec $POSTGRES_CONTAINER pg_dump -U moremong moremong | gzip > $BACKUP_DIR/postgres_$TIMESTAMP.sql.gz

# Redis 백업
docker exec moremong-redis redis-cli --rdb /data/dump.rdb
docker cp moremong-redis:/data/dump.rdb $BACKUP_DIR/redis_$TIMESTAMP.rdb

# 7일 이전 백업 삭제
find $BACKUP*DIR -name "postgres*_.sql.gz" -mtime +7 -delete
find $BACKUP*DIR -name "redis*_.rdb" -mtime +7 -delete

echo "백업 완료: $TIMESTAMP"
EOF
```

```
sudo chmod +x /opt/moremong/backup-db.sh
```

crontab에 추가 (매일 새벽 2시)

```
(crontab -l 2>/dev/null; echo "0 2 \* \* \* /opt/moremong/backup-db.sh >> /var/log/moremong-backup.log 2>&1") | crontab -
```

## 14단계: 헬스 체크 스크립트

```
sudo tee /opt/moremong/health-check.sh > /dev/null <<'EOF'
#!/bin/bash

echo "=== Moremong 헬스 체크 ==="
echo "타임스탬프: $(date)"
echo ""

# 서비스 확인
echo "백엔드 API 1:" $(systemctl is-active moremong-api-1)
echo "백엔드 API 2:" $(systemctl is-active moremong-api-2)
echo "프론트엔드 1:" $(systemctl is-active moremong-front-1)
echo "프론트엔드 2:" $(systemctl is-active moremong-front-2)
echo "Nginx:" $(systemctl is-active nginx)
echo "PostgreSQL:" $(docker ps --filter name=moremong-postgres --format "{{.Status}}")
echo "Redis:" $(docker ps --filter name=moremong-redis --format "{{.Status}}")
echo ""

# 포트 상태 확인
echo "=== 포트 상태 ==="
netstat -tuln | grep -E ':(80|443|5173|5174|8090|8095) '
echo ""

# 디스크 사용량 확인
echo "=== 디스크 사용량 ==="
df -h /
echo ""

# 메모리 사용량 확인
echo "=== 메모리 사용량 ==="
free -h
echo ""
EOF
```

```
sudo chmod +x /opt/moremong/health-check.sh
```

## 15단계: 배포 체크리스트

운영 전 이 체크리스트 실행

### 1. 모든 서비스 실행 확인

```
sudo systemctl status moremong-api-1 moremong-api-2
sudo systemctl status moremong-front-1 moremong-front-2
sudo systemctl status nginx
sudo docker compose -f /opt/moremong/docker/docker-compose.yml ps
```

### 2. 엔드포인트 테스트

```
curl -I http://localhost:8090/management/health
curl -I http://localhost:5173/worklog
curl -I https://moremong.com/worklog
curl -I https://moremong.com/api/management/health
```

### 3. 에러 로그 확인

```
sudo journalctl -u moremong-api-1 --since "1 hour ago" | grep -i error
sudo journalctl -u moremong-front-1 --since "1 hour ago" | grep -i error
sudo tail -n 100 /var/log/nginx/error.log
```

### 4.  SSL 확인

```
curl -I https://moremong.com
openssl s_client -connect moremong.com:443 -servername moremong.com < /dev/null
```

### 5.  OAuth 로그인 테스트

브라우저에서: https://moremong.com/worklog  
로그인 클릭, /worklog/oauth/callback으로 리다이렉트 확인

### 6.  방화벽 확인

```
sudo ufw status verbose
```

OCI 보안 목록에 80, 443 포트가 열려있는지 확인

### 7.  헬스 체크 실행

```
/opt/moremong/health-check.sh
```

### 8. 백업 테스트

```
/opt/moremong/backup-db.sh
ls -lh /opt/moremong/backups/
```

## 16단계: OAuth 제공자 업데이트

OAuth 제공자 콘솔에서 리다이렉트 URI 업데이트:

Google Cloud Console:

- 승인된 리다이렉트 URI: https://moremong.com/login/oauth2/code/google

Kakao Developers:

- Redirect URI: https://moremong.com/login/oauth2/code/kakao

Naver Developers:

- Callback URL: https://moremong.com/login/oauth2/code/naver

백엔드 application.yml 업데이트:

```
application:
  oauth2:
    allowed-redirect-uris:
      - https://moremong.com/worklog/oauth/callback
      - https://moremong.com/worklog1/oauth/callback
      - https://moremong.com/worklog2/oauth/callback
```

## 17단계: 성능 튜닝

프로덕션용 JVM 튜닝

/etc/systemd/system/moremong-api-1.service 편집  
ExecStart를 최적화된 플래그로 업데이트:

```
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
```

리로드 및 재시작

```
sudo systemctl daemon-reload
sudo systemctl restart moremong-api-1 moremong-api-2
```

PostgreSQL 튜닝

- docker-compose.yml에 PostgreSQL 튜닝 추가

```
sudo tee -a /opt/moremong/docker/docker-compose.yml <<'EOF'
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
EOF
```

PostgreSQL 재시작

```
cd /opt/moremong/docker
sudo docker compose restart postgresql
```

문제 해결 명령어

```
# 서비스 로그 확인
sudo journalctl -u moremong-api-1 -n 100 --no-pager
sudo journalctl -u moremong-front-1 -n 100 --no-pager
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# 서비스 재시작
sudo systemctl restart moremong-api-1 moremong-api-2
sudo systemctl restart moremong-front-1 moremong-front-2
sudo systemctl restart nginx

# 네트워크 연결 확인
sudo netstat -tuln | grep LISTEN
curl -I http://localhost:8090/management/health
curl -I http://localhost:5173/worklog

# Docker 로그
sudo docker compose -f /opt/moremong/docker/docker-compose.yml logs -f

# Nginx 테스트 및 리로드
sudo nginx -t
sudo nginx -s reload

# 디스크 공간 확인
df -h
sudo du -sh /opt/moremong/\*
sudo docker system df

# 프로세스 모니터링
htop
ps aux | grep java
ps aux | grep node

빠른 롤백 절차

# 모든 서비스 중지
sudo systemctl stop moremong-api-1 moremong-api-2
sudo systemctl stop moremong-front-1 moremong-front-2

# 백업에서 데이터베이스 복원
cd /opt/moremong/backups
LATEST*BACKUP=$(ls -t postgres*\*.sql.gz | head -1)
gunzip < $LATEST_BACKUP | docker exec -i moremong-postgres psql -U moremong -d moremong

# 이전 JAR/빌드 버전 배포
# (/opt/moremong/versions/에 버전별 백업 유지)
sudo cp /opt/moremong/versions/backend-v1.0.0.jar /opt/moremong/backend/moremong-restapi.jar

# 서비스 재시작
sudo systemctl start moremong-api-1 moremong-api-2
sudo systemctl start moremong-front-1 moremong-front-2
```
