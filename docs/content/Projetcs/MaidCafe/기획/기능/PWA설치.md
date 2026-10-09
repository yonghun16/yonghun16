---
title: PWA 설치
description: 키오스크 화면을 홈 화면에 설치 가능한 PWA로 제공하는 기능. 안드로이드/데스크탑은 매니페스트 기반 설치, iOS는 홈 화면 추가로 앱처럼 실행됨. 메뉴/주문 등 API 응답은 절대 캐시하지 않고 항상 최신 상태를 유지
tags: [기능, 프론트엔드, 완료]
aliases: []
created: 2026-09-10
updated: 2026-09-10
status: active
---

## 개요
"고객 화면을 앱 버전으로 만들고 싶다"는 요청에서 시작된 논의 끝에,
네이티브 앱(안드로이드/iOS)은 나중에 별도로 만들기로 하고 우선 지금
있는 Next.js 코드 그대로 설치 가능한 PWA(Progressive Web App)로
만들었습니다. [[개발계획서]]에 이미 정리돼 있던 "모바일 네이티브 앱
계획이 있으면 백엔드를 분리 유지한다"는 결론과도 맞아떨어져서, 백엔드
(Express, `apps/backend`)는 그대로 분리 유지합니다 — PWA는 별도
클라이언트가 아니라 지금 Next.js 앱을 설치 가능하게만 만드는 것이라
이 결론에 영향을 주지 않습니다.

## 동작 방식
- `app/manifest.ts`(Next.js App Router의 매니페스트 파일 컨벤션)가
  빌드 시 `/manifest.webmanifest`로 자동 생성되고 `layout.tsx`의
  `<head>`에 자동으로 링크됩니다. `start_url`을 `/`(고객 주문 화면)로
  지정해, 설치된 아이콘을 누르면 관리자/주방 화면이 아니라 항상 주문
  화면부터 열립니다.
- 아이콘은 기존 `app/icon.png`(파비콘용, 1254×1254 원본)를 192px/
  512px/마스커블(512px, 안전 영역 약 80%로 축소 후 배경색 채움) 세
  종류로 리사이즈해 만들었습니다. iOS Safari는 웹 매니페스트를
  부분적으로만 지원해서, `layout.tsx`에 `appleWebApp` 메타데이터
  (standalone 모드, 상태바 스타일)와 `app/apple-icon.png`(180×180,
  Next.js 파일 컨벤션)를 별도로 추가했습니다.
- 서비스워커는 `@serwist/next`(Next.js App Router 전용 PWA 빌드
  플러그인)로 구현했습니다. 더 널리 알려진 `next-pwa`는 App Router
  호환성 이슈로 관리가 뜸해진 편이라, App Router 대상으로 활발히
  유지되는 `@serwist/next`를 선택했습니다. 소스는 `app/sw.ts`, 빌드
  시 `public/sw.js`로 번들링됩니다(빌드 산출물이라 git에는 커밋하지
  않음 — `.gitignore` 처리). 서비스워커 등록 코드
  (`navigator.serviceWorker.register`)는 `@serwist/next`의 웹팩
  플러그인이 클라이언트 번들에 자동으로 주입해서, 별도로 등록하는
  컴포넌트를 만들 필요가 없습니다.
- **`/api/*` 요청(메뉴/카테고리/광고/주문/관리자 세션 — 전부
  `next.config.js`의 `rewrites()`가 백엔드로 중계)은 절대 캐시하지
  않습니다.** Serwist의 기본 캐싱 전략(`defaultCache`)이 API 응답을
  정확히 어떻게 다루는지 문서에서 명확히 확인되지 않아서, `NetworkOnly`
  규칙을 라우트 배열 맨 앞에 직접 추가해 API 요청이 정적 리소스용
  캐시 전략에 걸리지 않게 했습니다(Workbox 라우트는 배열 순서대로 첫
  매치를 씀) — 키오스크는 메뉴 품절/가격/재고가 실시간으로 바뀌어야
  하는 화면이라, 캐시된 옛 데이터를 보여주면 안 되기 때문입니다.
  이미지/폰트/CSS/JS 같은 정적 리소스만 `defaultCache`(캐시 우선류
  전략)를 그대로 씁니다.
- 개발 중에는 서비스워커가 핫리로드와 자주 충돌해 디버깅을 방해하므로,
  `NODE_ENV === 'development'`일 때는 서비스워커 자체를 만들지
  않습니다(`disable` 옵션). 프로덕션 빌드에서만 실제로 동작합니다.
- 오프라인 상태에서 주문을 큐에 쌓았다가 온라인이 되면 다시 보내는
  기능(백그라운드 동기화)은 이번 범위에 넣지 않았습니다 — 지금은
  "설치 가능한 화면" 정도가 목표라, 온라인 상태에서만 정상 동작합니다.

## 검증 관련 메모
빌드된 프로덕션 서버를 로컬에서 띄워 `/manifest.webmanifest`,
`/sw.js`, `/apple-icon.png`, `/icons/*.png`가 전부 정상 응답하는 것과
`layout.tsx`의 `<head>`에 매니페스트 링크/테마 컬러/애플 메타 태그가
제대로 나오는 것, 빌드된 번들에 `navigator.serviceWorker.register`
호출이 실제로 포함된 것까지 확인했습니다. 다만 실제 안드로이드/iPad
기기에서 "홈 화면에 추가" 흐름 자체(설치 프롬프트, 아이콘 표시,
standalone 실행)는 이 세션에 브라우저/기기 도구가 없어 직접 확인하지
못했습니다 — 실제 기기에서 한 번 확인해보는 것을 권장합니다.

## 관련 코드
- `apps/frontend/src/app/manifest.ts` (PWA 매니페스트)
- `apps/frontend/src/app/sw.ts` (서비스워커 소스 — API는
  `NetworkOnly`, 나머지는 `defaultCache`)
- `apps/frontend/src/app/layout.tsx` (`appleWebApp`/`viewport.themeColor`
  메타데이터)
- `apps/frontend/src/app/apple-icon.png` (iOS 홈 화면 추가용 180×180
  아이콘), `apps/frontend/public/icons/`(매니페스트용 192/512/마스커블
  아이콘)
- `apps/frontend/next.config.js` (`withSerwist`로 감싸서 빌드 시
  `app/sw.ts` → `public/sw.js` 번들링)
- `apps/frontend/.gitignore` (`public/sw.js`는 빌드 산출물이라 제외)
- `apps/frontend/tsconfig.json` (`sw.ts`는 `webworker`/`dom` lib 충돌
  때문에 `check-types` 대상에서 제외 — 빌드 시엔 `@serwist/next`가
  자체적으로 번들링함)

## 관련 문서
- [[기획서]]
- [[개발계획서]] — 백엔드를 Express로 분리 유지하기로 한 이유(네이티브
  앱 계획)
- [[09-프론트엔드-상태관리와-PWA]] — 서비스워커 캐싱 전략(`NetworkOnly`
  vs `defaultCache`)을 더 깊이 다루는 문서
