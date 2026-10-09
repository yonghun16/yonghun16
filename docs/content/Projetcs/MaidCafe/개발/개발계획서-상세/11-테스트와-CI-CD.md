---
title: 테스트 전략과 CI/CD
description: 백엔드는 인메모리 MongoDB로 실제 요청-응답을 검증하고, 프론트엔드는 순수 로직만 골라 테스트하는 이유, 그리고 GitHub Actions 파이프라인 구성
tags: [개발계획서, 테스트, CI, 백엔드, 프론트엔드]
aliases: []
created: 2026-09-12
updated: 2026-09-12
status: active
---

## TL;DR
백엔드는 **Vitest + Supertest + `mongodb-memory-server`**로 "실제
HTTP 요청 → Express 미들웨어 → MongoDB"까지 통째로 검증하는 통합
테스트 위주입니다. 프론트엔드는 컴포넌트 렌더링 테스트 없이 **순수
로직(Zustand 스토어, 유틸 함수)만** 테스트합니다. 둘 다 같은 Vitest를
쓰지만 검증 대상의 성격이 다릅니다. CI는 GitHub Actions로 push/PR마다
타입체크+빌드+테스트를 돌립니다.

## 왜 백엔드는 유닛 테스트가 아니라 통합 테스트 위주인가
`requireAdmin` 미들웨어, 세션 쿠키, Mongoose 쿼리를 각각 따로
모킹(mock)해서 테스트할 수도 있지만, 이 프로젝트는 **진짜 Express
앱 인스턴스에 진짜 HTTP 요청을 보내고, 진짜(인메모리) MongoDB에
읽고 쓰는** 방식을 택했습니다. 이유는 이 백엔드의 위험 요소 대부분이
"각 함수가 개별적으로 맞는지"가 아니라 **"미들웨어 순서, 세션 쿠키
전달, 쿼리 필터 조합이 실제로 맞물려 동작하는지"**에 있기 때문입니다
(예: `002-order-search-filter`에서 실제로 날짜+매장구분 조합 필터가
맞물려 동작하는지가 핵심 검증 대상이었음 — [[05-주문-핵심-알고리즘]]
4번 항목 참고). 모킹이 많아질수록 "테스트는 통과하는데 실제로는 안
되는" 괴리가 커질 위험도 있습니다.

## `createApp(sessionMongoUrl)` — 테스트를 위해 나눠둔 진입점

```ts
// src/app.ts — 앱 조립만, 서버 기동/DB 연결은 안 함
export function createApp(sessionMongoUrl: string): Express { ... }

// src/index.ts — 실제 서버 기동 담당
const app = createApp(process.env.MONGO_URI);
app.listen(PORT, ...);
```

`app.ts`가 `sessionMongoUrl`을 **파라미터로 받는 이유**가 테스트
설계의 핵심입니다. 운영에서는 `MONGO_URI`(Atlas)를 넘기고, 테스트에서는
`mongodb-memory-server`가 그때그때 띄운 인메모리 MongoDB의 주소를
넘깁니다 — **운영 DB를 절대 건드리지 않고도** 로그인 → 세션 저장 →
관리자 API 호출까지 실제 흐름 그대로 검증할 수 있습니다.

```ts
export async function setupTestApp() {
  const mongod = await MongoMemoryServer.create();
  await mongoose.connect(mongod.getUri());
  const app = createApp(mongod.getUri());
  return { app, teardown: async () => { /* DB 삭제, 연결 종료, 서버 정지 */ } };
}
```

각 테스트 파일(`describe` 블록)이 `beforeAll`에서 새 인메모리 DB를
띄우고 `afterAll`에서 완전히 정리합니다 — 테스트 파일 사이에 데이터가
남아 서로 영향을 주는 일이 없습니다. `loginAsAdmin(app)`은 Supertest의
`agent`(쿠키를 요청 사이에 자동으로 유지하는 클라이언트)로 실제
로그인 API를 호출해 인증된 세션을 만듭니다 — 세션 검증 로직 자체를
우회하지 않고 실제로 통과시킵니다.

## 환경변수를 어떻게 채우는가 — `setupEnv.ts`

`r2Client.ts`는 **모듈이 import되는 순간**(함수 호출이 아니라) 필수
환경변수를 확인하고 없으면 `process.exit(1)`합니다([[06-이미지-파이프라인]]
참고) — 이 말은 테스트 파일이 이 모듈을 import하기도 전에 이미
환경변수가 준비돼 있어야 한다는 뜻입니다. Vitest의 `setupFiles`
(`tests/setupEnv.ts`)는 각 테스트 파일의 import 자체가 실행되기
**전에** 먼저 돌아서, `ADMIN_PASSWORD`/`R2_*`/`VAPID_*` 같은 값에
전부 테스트용 더미 값을 채워둡니다. 실제 R2/VAPID 서비스와 통신하는
테스트(`webPush.test.ts`)는 이 더미 값 대신 `web-push` 모듈 자체를
모킹해서 네트워크 호출 없이 로직만 검증합니다.

## `fileParallelism: false` — 왜 병렬 테스트를 껐는가
`mongodb-memory-server`는 테스트가 처음 실행될 때 실제 MongoDB
바이너리를 로컬에 캐시로 내려받습니다. 테스트 파일을 병렬로 돌리면
여러 프로세스가 동시에 같은 캐시 경로에 파일을 내려받으려다
rename 작업이 서로 충돌하는 문제(알려진 이슈)가 있어서, 파일 단위
병렬 실행을 끄고 한 번에 하나씩만 바이너리 캐시에 접근하게 했습니다
— 테스트가 조금 느려지는 대신 안정적으로 통과합니다.

## 왜 프론트엔드는 컴포넌트 렌더링 테스트가 없는가
프론트엔드 테스트는 `features/cart/model/store.test.ts`,
`features/order-type/model/store.test.ts`,
`shared/lib/reorderArray.test.ts`,
`shared/lib/getErrorMessage.test.ts`처럼 **입력을 넣으면 정해진
출력이 나오는 순수 로직**만 골라 테스트합니다. 장바구니 병합 규칙
([[05-주문-핵심-알고리즘]] 3번 항목), 배열 재배치([[08-관리자-화면-UI-패턴]]
1번 항목) 같은, "이 함수가 이 입력에 대해 항상 같은 결과를 내는가"를
검증하기 좋은 대상들입니다.

반면 IntersectionObserver 기반 헤더 도킹([[09-프론트엔드-상태관리와-PWA]])
이나 실제 화면 클릭 흐름 같은 건 jsdom(Vitest의 기본 DOM 환경) 안에서
브라우저 API를 흉내내는 것보다 **실제 화면에서 눈으로 확인하는 게 더
정확하고 빠릅니다** — 이 프로젝트는 그런 시각적/상호작용 검증을 자동화
테스트 대신 개발 중 수동 확인(그리고 필요 시 화면 캡처로 기록)으로
대체하는 방침을 택했습니다.

## GitHub Actions — `ci.yml`

```yaml
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }

steps:
  - Setup pnpm / Node 22
  - Cache MongoDB binary (mongodb-memory-server가 매번 새로 받지 않게)
  - pnpm install --frozen-lockfile
  - pnpm turbo run check-types
  - pnpm turbo run build
  - pnpm turbo run test
```

- **별도 시크릿이 필요 없음**: `setupEnv.ts`가 필요한 모든 환경변수에
  더미 값을 채우고, MongoDB도 실제 Atlas가 아니라 인메모리로 뜨기
  때문에, CI 환경에 `R2_ACCESS_KEY_ID` 같은 실제 운영 비밀값을 등록할
  필요가 없습니다 — 저장소 시크릿 관리 부담이 없습니다.
- **`--frozen-lockfile`**: `pnpm-lock.yaml`과 `package.json`이 어긋나면
  (예: 로컬에서 lockfile을 안 커밋한 채 의존성만 추가) 설치 자체를
  실패시켜서, "로컬에서는 되는데 CI에서는 안 되는" 재현 불가능한 상태를
  방지합니다.
- **`turbo run <task>`**: 1장/2장에서 설명한 Turborepo 오케스트레이션을
  그대로 씁니다 — `packages/types`가 먼저 빌드된 뒤 이를 참조하는
  `apps/frontend`/`apps/backend`가 빌드되는 순서가 로컬과 CI에서
  동일하게 보장됩니다.
- **MongoDB 바이너리 캐시**: `mongodb-memory-server`가 받는 실제
  MongoDB 실행 파일을 GitHub Actions의 캐시로 저장해, CI를 돌릴
  때마다 몇십 MB를 다시 받는 걸 피합니다.

## `deploy.yml`과의 관계
`ci.yml`은 `main`에 대한 push/PR마다 품질 게이트로 동작하고,
실제 배포(Vercel/Render)는 각 플랫폼이 자체적으로 `main` 브랜치를
감시해 자동 배포하는 구조라 별도의 수동 배포 스텝은 두지 않았습니다
— 자세한 배포 파이프라인/인프라 설정은 [[12-배포-인프라]] 참고.

## 관련 문서
- [[개발계획서]]
- [[05-주문-핵심-알고리즘]] — 통합 테스트가 실제로 검증하는 로직들
- [[06-이미지-파이프라인]] — `setupEnv.ts`가 왜 필요한지의 배경
- [[12-배포-인프라]]
