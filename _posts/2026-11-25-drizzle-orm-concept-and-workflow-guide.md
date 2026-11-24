---
title: "Drizzle ORM 완벽 이해 — 개념부터 db:push가 필요한 이유, 전체 워크플로우까지"
excerpt: "왜 npm run db:push 같은 명령어를 매번 실행해야 할까? Drizzle ORM과 Drizzle Kit의 역할 분리, 코드(schema.ts)와 실제 DB가 동기화되는 원리, 전체 워크플로우, 그리고 MariaDB 환경에서 db:push TRUNCATE 오탐 시 db:migrate로 안전하게 전환하는 방법까지 정리합니다."
categories:
    - Development
    - Database
tags:
    - drizzle
    - drizzle-orm
    - drizzle-kit
    - orm
    - typescript
    - database
    - migration
    - mariadb
    - mysql
    - troubleshooting
    - guide
last_modified_at: 2026-11-25T01:35:00+09:00
toc: true
toc_sticky: true
---

프로젝트를 세팅하다 보면 `package.json`에 들어 있는 `npm run db:push`나 `npm run db:studio` 같은 명령어를 습관처럼 실행하게 됩니다. 하지만 막상 **"왜 굳이 이 명령어를 따로 쳐야 하지? 코드에서 테이블 정의했으면 그냥 알아서 생기는 거 아닌가?"**라는 의문이 들기 쉽습니다.

이 글에서는 **Drizzle ORM의 핵심 개념과 구조**, **`db:push` 같은 명령어가 필요한 이유**, **프로젝트 처음부터 운영 배포까지 이어지는 전체 워크플로우와 사용 방법**, 그리고 **MariaDB 환경에서 `db:push` 데이터 손실(TRUNCATE) 경고가 뜰 때 `db:migrate`로 안전하게 전환하는 실전 트러블슈팅**까지 한 번에 정리합니다.

---

### 1. 왜 `npm run db:push` 같은 명령어가 필요할까?

결론부터 말하면, **TypeScript 코드(`schema.ts`)는 설계도일 뿐이며, 실제 데이터베이스(SQLite, PostgreSQL 등) 내부의 테이블 구조까지 자동으로 바꿔주지는 않기 때문**입니다.

조금 더 풀어서 보면 다음과 같습니다.

1. 우리가 작성하는 `schema.ts`는 **"DB에 이런 테이블과 컬럼이 있다"고 TypeScript와 앱에 알려주는 설계도**입니다.
2. 이 파일을 수정했다고 해서 실제 DB 서버나 DB 파일(`.db`)에 `CREATE TABLE`이나 `ALTER TABLE` 같은 SQL이 자동으로 실행되지는 않습니다.
3. 만약 `schema.ts`에 새 컬럼을 추가해 놓고 `db:push`(또는 마이그레이션)를 실행하지 않으면, **코드 상에서는 자동완성과 타입 체크가 멀쩡히 통과하지만**, 막상 앱을 실행해 쿼리를 날리는 순간 실제 DB가 `"column does not exist"` 같은 런타임 에러를 발생시킵니다.

즉, **코드의 설계도(`schema.ts`)와 실제 데이터베이스 구조를 동기화(Sync)해 주는 다리 역할**을 하는 명령어가 바로 `db:push`, `db:generate`, `db:migrate` 같은 Drizzle CLI 명령어입니다.

---

### 2. Drizzle의 두 가지 축: `drizzle-orm` vs `drizzle-kit`

Drizzle은 역할을 철저히 두 패키지로 나누어 관리합니다. 이 둘의 차이를 알면 전체 구조가 훨씬 명확해집니다.

| 패키지 | 설치 위치 | 실행 시점 | 핵심 역할 |
| :--- | :--- | :--- | :--- |
| **`drizzle-orm`** | `dependencies` (런타임) | 웹/API 서버가 동작할 때 | `db.select()`, `db.insert()` 같은 **쿼리를 TypeScript로 작성해 SQL로 실행** |
| **`drizzle-kit`** | `devDependencies` (개발 도구) | 터미널에서 CLI로 실행할 때 | `schema.ts`를 읽어 **실제 DB 테이블을 생성/수정(`push`, `generate`, `migrate`)하거나 데이터를 조회(`studio`)** |

#### 왜 앱 실행 시 자동으로 테이블을 안 만들까?
일부 ORM이나 프레임워크는 앱이 부팅될 때마다 스키마를 검사해 테이블을 자동 수정하기도 합니다. 하지만 Drizzle은 **런타임(`drizzle-orm`)을 최대한 가볍고 빠르게 유지**하고, **DB 구조를 변경하는 위험한 작업(`DDL`)은 개발자가 CLI(`drizzle-kit`)를 통해 명시적으로 제어**하도록 설계되었습니다.

> **`npm run db:push`의 정체**  
> `npm run db:push`는 Drizzle 고유의 특수한 명령어가 아니라, 프로젝트의 `package.json` `"scripts"`에 등록된 **단축 별칭**입니다. 실제로는 내부에서 `drizzle-kit push`를 실행합니다.
>
> ```json
> {
>   "scripts": {
>     "db:push": "drizzle-kit push",
>     "db:generate": "drizzle-kit generate",
>     "db:migrate": "drizzle-kit migrate",
>     "db:pull": "drizzle-kit pull",
>     "db:studio": "drizzle-kit studio"
>   }
> }
> ```

---

### 3. 전체 동작 흐름: 상황에 따른 2가지 Sync 전략

Drizzle은 **Code-First(코드 우선)** 철학을 따릅니다. 즉, **TypeScript 스키마 파일(`schema.ts`)이 기준(Single Source of Truth)**이고, DB를 그 코드에 맞춥니다.

이때 개발 단계에 따라 DB에 변경사항을 반영하는 흐름이 크게 두 갈래로 나뉩니다.

#### 흐름 A: 빠른 로컬 개발 & 프로토타이핑 (`push` 방식)

```text
[schema.ts 수정] ──(drizzle-kit push)──> [실제 DB에 즉시 CREATE / ALTER 반영]
```

- **동작 원리:** `drizzle-kit`이 현재 `schema.ts`와 실제 DB 상태를 직접 비교한 뒤, 차이점만 즉시 SQL로 실행해 DB를 맞춥니다.
- **특징:** 중간에 마이그레이션 기록(`.sql` 파일)을 남기지 않고 한 번에 반영되므로 매우 빠르고 편합니다.
- **적합한 상황:** 혼자 빠르게 아이디어를 검증할 때, 로컬 SQLite/테스트 DB에서 테이블 구조를 자주 갈아엎을 때.
- **주의할 점:** 변경 히스토리가 파일로 남지 않기 때문에 운영 서버(Production) 배포나 팀 협업에서는 권장되지 않습니다.

#### 흐름 B: 실무 / 팀 협업 / 운영 배포 (`generate` + `migrate` 방식)

```text
[schema.ts 수정]
    │
    ▼  (1) npx drizzle-kit generate
[drizzle/0001_xxx.sql + 스냅샷 생성]  ──> Git 커밋으로 팀/배포 환경에 공유
    │
    ▼  (2) npx drizzle-kit migrate (또는 앱 부팅 시 코드로 실행)
[실제 DB에 미적용 SQL 순차 실행 + __drizzle_migrations 이력 기록]
```

- **동작 원리:**
  1. `generate`: 기존 스냅샷(`meta/` 폴더)과 바뀐 `schema.ts`를 비교해 **실행할 SQL 파일(`.sql`)**을 생성합니다. 이 단계에서는 실제 DB를 건드리지 않습니다.
  2. `migrate`: 실제 DB 안에 `__drizzle_migrations`라는 관리용 테이블을 두고, 아직 실행되지 않은 `.sql` 파일만 **순서대로 딱 한 번씩** 적용합니다.
- **특징:** 언제, 누가, 어떤 SQL로 DB를 바꿨는지 Git 이력으로 남고, 로컬/스테이징/운영 DB에 동일한 변경을 안전하게 재현할 수 있습니다.
- **적합한 상황:** 실제 서비스 운영, 팀 단위 협업, CI/CD 자동 배포 파이프라인.

#### 참고: 반대 방향 흐름 (`pull`, Database-First)

```text
[이미 존재하는 레거시 DB] ──(drizzle-kit pull)──> [schema.ts 자동 생성]
```

이미 테이블이 다 만들어져 있는 기존 프로젝트에 Drizzle을 도입하거나, 외부 툴에서 DB를 직접 수정했을 때 현재 DB 구조를 역추적(Introspect)해 `schema.ts` 코드를 뽑아내는 용도입니다.

---

### 4. `drizzle-kit` 주요 명령어 한눈에 보기

| 명령어 | 방향 / 대상 | 핵심 기능 | 주 사용처 |
| :--- | :--- | :--- | :--- |
| **`push`** | 코드 → 실제 DB | 마이그레이션 파일 없이 스키마 변경사항을 DB에 즉시 반영 | 로컬 개발, 프로토타이핑 |
| **`generate`** | 코드 → `.sql` 파일 | 스키마 변경 내역을 SQL 마이그레이션 파일과 스냅샷으로 생성 | Git 커밋 전 스키마 변경 기록 |
| **`migrate`** | `.sql` 파일 → 실제 DB | 아직 적용되지 않은 마이그레이션 SQL을 순서대로 실행 | 운영 배포, CI/CD, 팀 동기화 |
| **`pull`** | 실제 DB → 코드 | 기존 DB를 읽어 Drizzle용 `schema.ts` 파일 자동 생성 | 레거시 DB에 Drizzle 도입 |
| **`check`** | 마이그레이션 파일 검증 | 브랜치 병합 등으로 마이그레이션 파일 순서/스냅샷이 꼬이지 않았는지 검사 | PR / CI 검증 단계 |
| **`studio`** | 실제 DB → 웹 UI | 브라우저 기반 DB 뷰어를 띄워 테이블과 데이터를 조회·수정 | 개발 중 DB 데이터 확인 |

---

### 5. 처음부터 끝까지: Drizzle 전체 사용 방법

PostgreSQL(`postgres` 드라이버)을 기준으로, 프로젝트 세팅부터 실제 쿼리 실행까지의 전체 과정입니다. *(SQLite나 MySQL도 드라이버와 `pg-core` 대신 `sqlite-core` / `mysql-core`를 쓰는 것 외에는 사용 흐름이 동일합니다.)*

#### Step 1. 패키지 설치

런타임용 `drizzle-orm` + DB 드라이버와, 개발용 `drizzle-kit`을 구분해 설치합니다.

```bash
# 런타임 의존성
npm install drizzle-orm postgres

# 개발 의존성 (CLI 도구 및 TypeScript 실행기)
npm install -D drizzle-kit tsx dotenv
```

#### Step 2. `.env` 및 `drizzle.config.ts` 작성

프로젝트 루트에 DB 접속 주소(`.env`)와 **`drizzle-kit`이 참고할 지도(`drizzle.config.ts`)**를 작성합니다.

```env
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

```typescript
// drizzle.config.ts
import 'dotenv/config';
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'postgresql',          // 사용할 DB 종류 ('postgresql' | 'mysql' | 'sqlite' 등)
  schema: './src/db/schema.ts',   // 설계도(스키마) 파일 경로
  out: './drizzle',               // generate 시 SQL 마이그레이션 파일이 저장될 폴더
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

> **`drizzle.config.ts`가 왜 중요할까?**  
> 우리가 `npm run db:push`를 치면, `drizzle-kit`은 제일 먼저 루트 경로의 `drizzle.config.ts`를 찾아 **"어떤 DB인지(`dialect`), 설계도는 어디 있는지(`schema`), 접속 주소는 무엇인지(`dbCredentials`)"**를 파악합니다.

#### Step 3. 스키마 정의 (`src/db/schema.ts`)

테이블 구조와 관계, 그리고 코드에서 사용할 TypeScript 타입을 정의합니다.

```typescript
// src/db/schema.ts
import { pgTable, serial, text, integer, timestamp } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// 1. users 테이블 정의
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// 2. posts 테이블 정의 (외래키 참조 포함)
export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  authorId: integer('author_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
});

// 3. Relational Query(db.query)를 위한 관계 정의 (선택 사항)
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, {
    fields: [posts.authorId],
    references: [users.id],
  }),
}));

// 4. 스키마로부터 TypeScript 타입 자동 추론
export type User = typeof users.$inferSelect;       // 조회(SELECT) 시 반환 타입
export type NewUser = typeof users.$inferInsert;    // 삽입(INSERT) 시 입력 타입
```

`$inferSelect`와 `$inferInsert`를 사용하면 별도의 `interface User`를 중복 작성할 필요 없이, 스키마를 바꿀 때마다 앱 전체 타입이 자동으로 따라옵니다.

#### Step 4. DB 연결 객체 생성 (`src/db/index.ts`)

애플리케이션 코드가 실제로 사용할 `db` 인스턴스를 만듭니다.

```typescript
// src/db/index.ts
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const client = postgres(process.env.DATABASE_URL!);

// 두 번째 인자로 { schema }를 넘겨줘야 db.query.* 관계형 쿼리를 사용할 수 있음
export const db = drizzle(client, { schema });
```

#### Step 5. `package.json` 스크립트 등록 및 DB 반영

자주 쓰는 명령어를 `package.json`에 등록합니다.

```json
{
  "scripts": {
    "db:push": "drizzle-kit push",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:studio": "drizzle-kit studio"
  }
}
```

이제 상황에 맞게 DB에 테이블을 생성합니다.

- **로컬에서 빠르게 바로 반영할 때:**
  ```bash
  npm run db:push
  ```
- **마이그레이션 파일로 관리하며 반영할 때:**
  ```bash
  npm run db:generate   # ./drizzle 폴더에 0000_xxx.sql 생성
  npm run db:migrate    # 생성된 SQL을 실제 DB에 적용
  ```
- **브라우저 UI로 테이블과 데이터를 눈으로 확인하고 싶을 때:**
  ```bash
  npm run db:studio
  ```

#### Step 6. 애플리케이션 코드에서 CRUD 쿼리 실행 (`src/index.ts`)

이제 실제 DB에 테이블이 만들어졌으므로 `db` 객체를 이용해 데이터를 다룰 수 있습니다. Drizzle은 SQL 문법과 거의 1:1로 대응되는 **Core Query API**와, 중첩된 관계 데이터를 쉽게 가져오는 **Relational Query API**를 모두 제공합니다.

```typescript
// src/index.ts
import { db } from './db';
import { users, posts } from './db/schema';
import { eq } from 'drizzle-orm';

async function main() {
  // 1. INSERT (생성 후 바로 결과 반환)
  const [newUser] = await db
    .insert(users)
    .values({
      name: '홍길동',
      email: 'hong@example.com',
    })
    .returning();

  await db.insert(posts).values({
    title: 'Drizzle 첫 글',
    content: 'SQL과 닮아서 직관적입니다.',
    authorId: newUser.id,
  });

  // 2. SELECT (SQL 스타일 Core API)
  const foundUsers = await db
    .select()
    .from(users)
    .where(eq(users.email, 'hong@example.com'));

  // 3. Relational Query (JOIN 없이 관계 데이터를 한 번에 객체로 조회)
  const usersWithPosts = await db.query.users.findMany({
    with: {
      posts: true,
    },
  });

  // 4. UPDATE
  await db
    .update(users)
    .set({ name: '김철수' })
    .where(eq(users.id, newUser.id));

  // 5. DELETE
  await db.delete(users).where(eq(users.id, newUser.id));

  console.log({ foundUsers, usersWithPosts });
}

main().catch(console.error);
```

테스트 실행은 `tsx`로 바로 돌려볼 수 있습니다.

```bash
npx tsx src/index.ts
```

---

### 6. 자주 헷갈리는 포인트 & 실전 팁

1. **`schema.ts`를 수정했는데 앱에서 에러가 나요!**
   - `drizzle-orm`은 설계도만 바뀐 상태에서 쿼리를 날려버립니다. 컬럼을 추가/수정했다면 반드시 `npm run db:push`(로컬) 또는 `npm run db:generate` + `npm run db:migrate`(운영)를 실행해 **실제 DB 구조도 함께 맞춰야** 합니다.

2. **`db:push` 실행 시 기존 데이터가 날아갈 수도 있나요?**
   - 네. 컬럼을 삭제하거나 타입을 호환되지 않는 형태로 바꿀 때 `drizzle-kit push`는 기존 데이터를 잃을 수 있음을 터미널 프롬프트로 경고합니다. 이때 **`Yes`를 선택하면 실제로 테이블이 `TRUNCATE`되거나 재생성되어 데이터가 삭제**되므로, 예상치 못한 경고가 뜨면 우선 **`No, abort`**를 선택하세요. 특히 MariaDB에서는 실제 타입이 같아도 오탐으로 `TRUNCATE`를 시도할 수 있으므로(아래 7번 항목 참고), 중요한 데이터가 있는 DB에서는 `generate`로 `.sql` 파일을 검토한 뒤 `migrate`를 실행하는 것이 안전합니다.

3. **`drizzle-kit`이 `.env`의 `DATABASE_URL`을 못 읽는다는 에러가 나요.**
   - Next.js 같은 프레임워크는 `.env.local`을 자체적으로 로드하지만, `drizzle-kit`은 프레임워크 밖에서 독립적으로 실행되는 CLI입니다. `drizzle.config.ts` 최상단에 `import 'dotenv/config'`를 넣거나(필요 시 `dotenv.config({ path: '.env.local' })` 지정), CLI 실행 시 환경변수를 명시적으로 전달해야 합니다.

4. **로컬에서는 `db:push`로 편하게 개발하다가 나중에 배포할 때는 어떻게 하나요?**
   - 초기 프로토타이핑이 어느 정도 안정화되면 로컬 개발 중이던 스키마를 기준으로 `npx drizzle-kit generate`를 실행해 초기 마이그레이션 `.sql` 파일을 만들고, 이후부터는 `generate` + `migrate` 워크플로우로 전환해 배포 파이프라인에 태우는 것이 정석입니다.
   - 단, 이미 `db:push`로 테이블이 생성된 DB에 곧바로 `db:migrate`를 실행하면 초기 SQL(`0000_xxx.sql`)을 다시 실행하려다 충돌이 나므로, `__drizzle_migrations`에 초기 이력(Baseline)을 먼저 등록해야 합니다(아래 7번 항목 참고).

---

### 7. 실전 트러블슈팅: MariaDB의 `db:push` TRUNCATE 오탐과 `db:migrate` 안전 전환

로컬이나 운영 DB로 **MariaDB**(예: MariaDB 11.x)를 사용하면서 `drizzle.config.ts`에 `dialect: 'mysql'`을 설정했을 때, 기존 데이터가 들어 있는 테이블을 두고 `npm run db:push`가 갑자기 테이블 삭제 후 재생성(`TRUNCATE`) 경고를 띄우는 현상이 발생할 수 있습니다.

#### 1) 증상: `TRUNCATE` 경고가 떴다면 반드시 `No, abort` 선택

`drizzle-kit push`가 터미널에서 데이터 손실 가능성을 경고하며 실행 여부를 물을 때:

- **`Yes`**를 선택하면 경고 대상 테이블에 대해 **실제로 `TRUNCATE` / 삭제 후 재생성이 실행되어 기존 데이터가 모두 사라집니다.**
- 의도적으로 테이블을 비울 계획이 아니라면 **무조건 `No, abort`를 선택**해 중단한 뒤, 왜 `drizzle-kit`이 이런 판단을 내렸는지부터 확인해야 합니다.

#### 2) 원인: MariaDB 표시 폭 메타데이터(`bigint(20) unsigned` vs `bigint unsigned`)

`schema.ts`의 실제 저장 타입을 바꾸지 않았는데도 `db:push`가 타입 변경으로 오판하는 주된 원인은 **MySQL 8과 MariaDB의 `INFORMATION_SCHEMA` 반환 형식 차이**입니다.

- **MariaDB:** 메타데이터(`INFORMATION_SCHEMA`)에 `bigint(20) unsigned`, `int(11)`처럼 **정수 표시 폭(display width)을 포함해 반환**합니다.
- **Drizzle Kit (`dialect: 'mysql'`):** 표시 폭 표기가 제거된 **MySQL 8 기준 타입 문자열(`bigint unsigned`)**을 기대하고 DB 메타데이터와 문자열 비교를 수행합니다.
- **결과:** DB 내부의 실제 바이너리 저장 타입은 완전히 동일한데도 `drizzle-kit push`가 이를 컬럼 타입 변경으로 오탐(False Positive)하면서, 이미 데이터가 들어 있는 테이블을 삭제 후 재생성(`TRUNCATE`)하려고 시도합니다.

#### 3) 해결: MariaDB에서는 `db:generate` → `db:migrate` 워크플로우 사용

MariaDB 환경에서는 매번 DB 메타데이터를 실시간으로 역비교(Introspection diff)하는 `db:push` 대신, `drizzle-kit generate`가 생성한 명시적 마이그레이션 SQL(`drizzle/0001_xxx.sql`, `drizzle/0002_xxx.sql` 등)을 순서대로 실행하는 **`npm run db:migrate`** 방식을 사용해야 합니다.

#### 4) 기존 데이터를 유지한 채 `db:migrate`로 전환하는 절차 (Baseline 등록)

이미 `db:push`(또는 수동 SQL)로 초기 스키마에 해당하는 테이블들이 DB에 생성되어 있고, 마이그레이션 이력 테이블(`__drizzle_migrations`)만 없는 상태라면 아래 순서로 안전하게 전환할 수 있습니다.

1. **현재 스키마와 마이그레이션 파일 확인**  
   - 초기 테이블 구성에 해당하는 `drizzle/0000_xxx.sql`과 이후 변경분(`0001_xxx.sql`, `0002_xxx.sql` 등)이 준비되어 있는지 확인합니다.
   - 이후 변경분 SQL에 의도치 않은 `DROP`이나 `TRUNCATE`가 없는지 파일 내용을 직접 눈으로 검증합니다.

2. **`__drizzle_migrations`에 `0000` Baseline 이력 1건 먼저 등록**  
   - 바로 `npm run db:migrate`를 실행하면 `0000_xxx.sql`의 `CREATE TABLE`을 다시 실행하려다 이미 존재하는 테이블과 충돌합니다.
   - 따라서 `__drizzle_migrations` 테이블에 **`0000` 마이그레이션이 이미 적용되었다는 기준점(Baseline) 이력 1건**을 먼저 기록해 줍니다.
   - MySQL/MariaDB용 Drizzle 마이그레이터는 `__drizzle_migrations` 테이블의 `created_at`과 `hash`를 `drizzle/meta/_journal.json`의 `entries[i].when` 값 및 해당 `.sql` 파일의 SHA-256 해시와 대조해 실행 여부를 판단합니다.

   ```sql
   -- 1) 이력 테이블이 아직 없다면 생성 (Drizzle 기본 추적 테이블 구조)
   CREATE TABLE IF NOT EXISTS `__drizzle_migrations` (
     `id` serial PRIMARY KEY,
     `hash` text NOT NULL,
     `created_at` bigint
   );

   -- 2) 0000 초기 마이그레이션을 Baseline으로 선등록
   --    hash: 0000_xxx.sql 파일 내용의 SHA-256 해시
   --    created_at: drizzle/meta/_journal.json 내 0000 항목의 "when" 타임스탬프
   INSERT INTO `__drizzle_migrations` (`hash`, `created_at`)
   VALUES ('<0000_sql_sha256_hash>', 1732893263399);
   ```

3. **`npm run db:migrate` 실행**  
   - Baseline 등록 후 `npm run db:migrate`를 실행하면 `0000`은 건너뛰고, 미적용 상태인 `0001`(미사용 테이블 정리 등)과 `0002`(신규 테이블·외래키·인덱스 생성 등)만 **기존 테이블의 데이터를 그대로 보존한 채** 순차 적용됩니다.

#### 5) 마이그레이션 도중 중단됐을 때의 복구 및 최종 검증

MySQL/MariaDB에서는 `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX` 같은 DDL 구문이 실행 즉시 암묵적으로 커밋(Implicit Commit)됩니다. 따라서 하나의 마이그레이션 파일 안에 여러 구문이 들어 있을 때 뒤쪽 구문에서 오류가 나면, **앞선 테이블과 외래키는 생성되었는데 인덱스 일부는 누락되고 `__drizzle_migrations`에는 기록이 남지 않은 부분 적용 상태**가 될 수 있습니다.

이런 경우에는 다음 절차로 상태를 맞춘 뒤 검증합니다.

1. **실제 DB 상태 점검 및 누락 객체 보완:** `SHOW TABLES`, `SHOW CREATE TABLE`, `SHOW INDEX` 등으로 실제 반영 여부를 확인하고, 중간 중단으로 빠진 인덱스나 제약조건이 있다면 SQL로 보완합니다.
2. **`__drizzle_migrations` 이력 동기화:** 실제 DB에 모두 반영이 완료된 마이그레이션(`0000`, `0001`, `0002` 등)과 `__drizzle_migrations` 테이블의 레코드가 일치하도록 맞춥니다.
3. **기존 데이터 보존 및 CLI 재검증:** 기존 테이블의 행 수(`SELECT COUNT(*)`)가 그대로 유지되었는지 확인하고, 마지막으로 `npm run db:migrate`를 다시 실행해 정상 완료 메시지를 확인합니다.

   ```bash
   npm run db:migrate
   # [✓] migrations applied successfully! 확인
   ```

> **MariaDB 프로젝트 운영 원칙**  
> 앞으로 스키마를 변경할 때도 MariaDB의 표시 폭 차이(`bigint(20) unsigned` vs `bigint unsigned`)로 인한 `db:push` 오탐 및 실수로 인한 데이터 유실을 막기 위해, 로컬과 운영 모두 **`npm run db:generate` → SQL 검토 → `npm run db:migrate`** 흐름을 일관되게 사용하는 것을 권장합니다.

---

### 요약
- `drizzle-orm`은 **앱 실행 중 SQL 쿼리를 날리는 엔진**이고, `drizzle-kit`은 **설계도(`schema.ts`)와 실제 DB 테이블 구조를 동기화하는 CLI 도구**입니다.
- `npm run db:push`가 필요한 이유는 **코드의 설계도를 바꿨을 때 실제 DB에도 `CREATE / ALTER TABLE`을 반영해야 앱이 정상 동작하기 때문**입니다.
- **로컬 빠른 개발은 `push`**, **팀 협업 및 운영 배포는 `generate` → `migrate`**, **데이터 확인은 `studio`**로 구분해 사용하면 깔끔합니다.
- 특히 **MariaDB** 환경에서는 표시 폭 메타데이터 차이로 `db:push`가 `TRUNCATE`를 오탐으로 시도할 수 있으므로, 경고 시 **`No, abort`**를 선택한 뒤 **`__drizzle_migrations` Baseline 등록 + `db:generate` → `db:migrate`** 흐름으로 안전하게 전환·관리하세요.
