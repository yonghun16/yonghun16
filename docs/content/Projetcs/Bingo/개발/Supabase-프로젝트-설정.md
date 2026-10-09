---
title: Supabase 프로젝트 설정
tags: [가이드]
description: 빙고 게임을 실제로 실행해보기 위해 Supabase 프로젝트를 만들고 .env를 채우는 절차
created: 2026-09-20
updated: 2026-09-30
status: active
---

로컬에서 `npm run dev`로 빙고 게임을 실제로 플레이해보려면 Supabase 프로젝트 1개와 `.env` 값만 있으면 된다. 이 프로젝트는 Broadcast + Presence만 쓰고 DB 테이블은 전혀 쓰지 않으므로, 테이블 생성이나 Replication 설정 없이 프로젝트 생성 → API 키 복사 → `.env` 작성이 전부다.

## 1. 프로젝트 생성

1. [supabase.com](https://supabase.com)에 가입/로그인한다.
2. "New Project"로 새 프로젝트를 만든다.
   - 조직(Organization) 선택 또는 새로 생성
   - 프로젝트 이름: 아무거나 (예: `bingo`)
   - Database Password: 자동 생성 값 사용해도 무방 (이 프로젝트는 Postgres를 직접 쓰지 않는다)
   - Region: 사용자와 가까운 리전 선택 (지연시간에 약간 영향, 실사용엔 큰 차이 없음)
3. 무료(Free) 플랜으로 충분하다 — 동시 Realtime 연결 200개, 메시지 처리량 제한이 있지만 최대 5~6인이 같이 하는 캐주얼 게임에는 넉넉하다.

## 2. API 키/URL 확보

> Supabase 대시보드는 메뉴 위치와 키 이름을 종종 바꾼다(예: "anon public" 키가 "Publishable key"로, URL 항목이 "Integrations → Data API"로 이동한 적이 있었다). 아래 메뉴명이 안 보이면 **Settings** 아래에서 "API"/"API Keys"가 들어간 항목을 찾아보면 된다 — 찾아야 하는 값 자체(공개해도 되는 URL + 공개용 키)는 바뀌지 않는다.

1. 프로젝트 대시보드에서 **Settings → API** (또는 **API Keys**)로 이동한다.
2. 아래 두 값을 복사해둔다.
   - **Project URL**(또는 Data API URL) → `.env`의 `VITE_SUPABASE_URL`
   - **anon public** 키(또는 **Publishable key**) → `.env`의 `VITE_SUPABASE_ANON_KEY`
3. **`service_role`(또는 Secret) 키는 절대 쓰지 않는다** — 서버 전용 관리자 키로, 클라이언트(브라우저) 코드에 넣으면 안 된다. 이 프로젝트는 공개용(anon/publishable) 키만으로 동작한다.

## 3. 별도 Realtime 설정은 필요 없다

- `src/features/bingo-room/api/roomChannel.ts`에서 채널을 `private: true`로 만들지 않았으므로, Realtime Authorization(RLS) 정책을 따로 설정할 필요가 없다. `anon` 키만으로 바로 연결된다.
- Postgres Changes(테이블 변경 구독)를 쓰지 않으므로 Database → Replication 설정도 건드릴 필요가 없다.

## 4. `.env` 작성

저장소 루트에서:

```bash
cp .env.example .env
```

`.env`를 열어 값을 채운다:

```
VITE_SUPABASE_URL=https://xxxxxxxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOi...
```

`.env`는 `.gitignore`에 이미 등록돼 있어 커밋되지 않는다.

## 5. 실행 및 확인

```bash
npm install
npm run dev
```

브라우저 탭 여러 개(또는 시크릿 창)로 같은 방 링크(`http://localhost:5173/room/아무값`)를 열어 서로 다른 닉네임으로 입장해보면 실제로 함께 플레이해볼 수 있다.

확인해볼 만한 시나리오는 [[개발프로세스]]의 "통합 테스트 체크리스트"와 각 스펙(`docs/content/specs/001-room-lifecycle` 등)의 Test 섹션을 참고한다.

## 관련 문서

- [[개발프로세스]]
- [[게임흐름]]
