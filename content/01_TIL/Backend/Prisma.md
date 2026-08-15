---
date: 2025-11-20
tags:
  - til
  - backend
  - prisma
---
# ❓ Information
* prisma 사용법 공부

---

# 🔰 Content ->  
## Supabase:
OpenSourse backend service

Database: Full PostGres DB
Authentication
Storage: S3과 호환되는 파일 저장소
Edge Function: 사용자와 가까운 엣지에서 배포되는 서버측 TS 함수
Realtime: 전세계적으로 분산된 Realtime 서버 클러스터
AI & Vectors: Postgres 및 pgVector 를 사용해 AI 애플리케이션을 개발하기 위한 오픈소스 툴킷

## Prisma:
DB 스키마를 쉽게 정의하고 Type-Safe 한 쿼리를 작성할 수 있도록 도와주는 차세대 Node.js 및 TS ORM 

직접 SQL을 작성하지 않고도 DB 를 쉽게 다룰수 있다
```ts
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()
prisma.user.create({
	data: {
		name: 'HEROPY',
		age: 85,
		email: 'thesecon@gmail.com'
	}
})
```


## SetUp
### lib
```bash
pnpm add -D prisma
pnpm add @prisma/clinet
```
- `prisma`
Prisma CLI 를 사용해 DB 스키마를 가져오거나 migration 실행하는 코어 패키지
- `@prisma/client` 
클라이언트 라이브러리로, DB에 대한 Type-Safe 한 쿼리를 요청할 수 있도록 도와줌


### 초기화
prisma 사용전 초기화
```bash
pnpm prisma init
```
초기 생성 당시에는 `schema.prisma` 에 default provider 가 sqlite로 되어 있을 거라 PostgreSQL로 바꿔야 함

### Supabase 플랫폼 설정
1. 프로젝트 생성 
Dashboard > Projects 

2. Connection 복사
Project root 에서 .env 파일에 비밀번호 설정
```ini
DATABASE_URL = "..."
...
```

3. Prisma에서 Supabase URL 사용 가능하도록 
schema.prisma 파일 수정
```ini
datasource db {
	provider = "postgresql"
	url = env("DATABASE_URL")
	directUrl = env("DIRECT_URL")
}
```

## Prisma 를 스키마 관리 도구로 삼아 Supabase DB를 설계/관리
Prisma가 스키마의 단일 출처 (SSOT) 가 되는 방식

Prisma -> Supabase 단방향 구조
- `schema.prisma` 파일이 DB 구조의 절대 기준이 됨
- **모든 변경은 `schema.prisma` 를 수정 -> `prisma migrate dev` 또는 `prisma migrate deploy`로 DB 반영**
- Supabase Dashboard 에서 임의로 컬럼/타입을 수정하면 로컬 스키마와 불일치 발생

대부분의 프로젝트는 이 구조를 기반으로 하고 
**Supabase 시스템 테이블 (auth, storage 등)은 `prisma db pull`로 가져와서 `schema.prisma` 자동 생성**

`schema.prisma`의 datasource 수정
```
datasource db {
	provider = "postgresql"
	url = env("DATABASE_URL")
}
```

Prisma 로 스키마 설계하고 적용하기 
```
model User {
	id        Int      @id @default(autoincrement())
    email     String   @unique
    createdAt DateTime @default(now())
}
```

DB에 반영
```
pnpm prisma migrate dev --name init
```

## Supabase 의 시스템 테이블도 함께 쓰고 싶을 때 (Supabase -> Prisma)
`auth.users` 테이블을 Prisma scheme에 추가하고 싶다면
```bash
pnpm prisma db pull
```
그럼 Supabase 시스템 테이블들이 전부 자동 생성된다
**하지만 auth  스키마 구조는 Prisma 에서 직접 변경하면 안된다**
-> Supabase 는 auth 관련 테이블을 관리하는 자체 로직이 있기 때문

그러므로 일반적으로 아래처럼 `schema.prisma` 파일을 두 파트로 분리하는 느낌으로 관리함

Prisma에서 직접 관리하는 모델
- app 전용 테이블
- prisma migrate 로 생성 및 수정

Supabase 가 자동 관리하는 테이블 (auth, storage, realtime 등)
- prisma db pull로 읽기 전용 모델
- 수정하지 않음

## Prisma Client 로 Supabase DB에 접근
적용 후 Prisma Client 생성
```bash
pnpm prisma generate
```

사용:
```ts
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()

const user = await prisma.user.create({
	data: { email: "test@email.com" }
})
```
Supabase의 Row Level Securiy(RLS)와도 쓸 수 있는데,
이 경우에는 Supabse의 Service_role 키를 백엔드에서 사용하면 모든 정첵을 우회할 수 있다

## 권장 패턴
✔ 애플리케이션 스키마: Prisma migrate  
✔ auth/storage/realtime 등 Supabase 제공 스키마: prisma db pull 로 읽기 전용  
✔ RLS 정책: Supabase 정책 시스템을 그대로 사용  
✔ Prisma는 ORM + Migration 중심 역할


## pnpm prisma generate 의 역할
TypeSafe 기능은 schema.prisma -> TS 및 Prisma Client 코드로 변환하는 과정이 필요함

```
schema.prisma (모델 정의)
|
v
pnpm prisma generate
|
v
node_modules/@prisma/client (코드 + 타입 생성됨)
|
v
import { PrismaClient } from '@prisma/client'
```

generate 실행 전
- `PrismaClient`는 예전 shcema 기준
- `prisma.user.findMany()` 같은 함수에서 타입 에러 발생하거나 
- 새로운 필드 인식하지 못함

generate 생성 후
- 새로운 모델이 타입으로 생성됨
- `prisma.user.create({ data: {email: ... }})`가 완전히 타입 세이프하게 적용됨
- TypeScript 자동완성 적용

정리하면 **generate 는 schema를 타입으로 변환해 실제로 쓸 수 있게 만드는 단계**
Prisma Client는 자동 생성되는 라이브러리이고, generate는 그 라이브러리를 최신 schema 기준으로 빌드하는 과정이다

언제 실행해야 하나
- schema.prisma 수정
- migrate(dev 또는 deploy)
- grnerate

Supabase에서 db pull을 실행한 후에도 generate 를 다시 해줘야 새로 생긴 모델을 타입 구조로 사용할 수 있다

