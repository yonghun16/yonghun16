---
title: 프론트엔드 상태 관리와 PWA
description: Zustand 스토어를 언제 어떻게 나누는지, IntersectionObserver로 만든 카테고리 헤더 도킹 효과, 서비스워커 캐싱 전략을 정리한 문서
tags: [개발계획서, 알고리즘, 프론트엔드, 상태관리]
aliases: []
created: 2026-09-12
updated: 2026-09-12
status: active
---

## TL;DR
이 프로젝트의 Zustand 스토어는 전부 **"화면 하나 또는 기능 하나
전용"**이고, 여러 화면이 공유해야 하는 순간에만 더 아래 FSD
레이어(entities)로 스토어를 옮깁니다. 이 장의 하이라이트는 카테고리
탭이 "스크롤하면 상단 고정 헤더 안으로 빨려 들어가는" 효과를
`IntersectionObserver` + 공유 스토어로 구현한 부분입니다.

## Zustand를 쓰는 기준 — "이 상태를 누가 알아야 하는가"
이 프로젝트에서 상태를 어디에 둘지는 항상 "이 상태를 몇 개의
컴포넌트/화면이 함께 봐야 하는가"로 결정합니다.

| 상태 | 위치 | 왜 |
|---|---|---|
| 장바구니(`items`, `totalPrice`) | `features/cart` (Zustand) | 상품 카드, 장바구니 패널, 헤더 배지 등 서로 다른 위치의 여러 컴포넌트가 동시에 읽고 씀 |
| 매장/포장 선택(`orderType`) | `features/order-type` (Zustand) | 화면 전환 조건(`HomePage`)과 주문 제출 양쪽이 필요 |
| 카테고리 필터 + 헤더 도킹 상태 | `entities/category` (Zustand) | 아래 "헤더 도킹" 절 참고 — 두 군데(원래 위치/헤더)가 같은 상태를 봐야 함 |
| 관리자 화면의 카테고리/상품/광고 목록 | `features/*-management` (Zustand) | 목록 조회 + CRUD + 드래그 재배치가 한 화면 안에서 여러 하위 컴포넌트에 걸쳐 있음 |
| 모달 열림/닫힘, 입력 폼 값 | 컴포넌트 로컬 `useState` | 그 컴포넌트(와 직계 자식) 밖에서는 아무도 알 필요 없음 |

**전역 상태로 승격하는 기준**은 FSD의 "코드 배치 판단 기준"과
동일합니다 — 처음엔 로컬 `useState`로 시작하고, "다른 컴포넌트도
이 값이 필요하다"는 게 드러나는 순간 그 상태가 필요한 컴포넌트들의
**가장 가까운 공통 조상에 해당하는 FSD 레이어**로 스토어를
옮깁니다. 카테고리 도킹 상태가 `widgets/product-list` 안이 아니라
`entities/category`에 있는 이유가 정확히 이것입니다(아래 참고).

## 카테고리 헤더 도킹 효과 — `IntersectionObserver`

**요구사항**: 평소엔 카테고리 탭이 메뉴 제목 바로 아래(원래 자리)에
있다가, 스크롤해서 이 탭이 상단 고정 헤더 밑으로 넘어가려는 순간
"탭이 헤더 안으로 빨려 들어가는" 것처럼 보여야 합니다.

### 왜 `scroll` 이벤트 리스너 + `scrollY` 비교가 아닌가
가장 먼저 떠오르는 방법은 `window.addEventListener('scroll', ...)`로
스크롤 위치를 계속 재고, 특정 픽셀 값을 넘으면 상태를 바꾸는
것입니다. 하지만 이 방식은 (1) 스크롤 이벤트가 초당 수십~수백 번
발생해 매번 콜백이 실행되고(성능), (2) "헤더 높이", "탭의 원래
위치" 같은 값을 픽셀 단위로 하드코딩해야 해서 반응형 레이아웃에서
쉽게 어긋납니다.

### `IntersectionObserver`가 더 잘 맞는 이유
`IntersectionObserver`는 "이 엘리먼트가 뷰포트(혹은 지정한 영역)와
겹치는지"를 브라우저가 최적화된 방식으로 감시해 **변화가 있을
때만** 콜백을 호출합니다. 스크롤마다 계산할 필요가 없습니다.

```ts
const HEADER_HEIGHT_PX = 56;

useEffect(() => {
  const target = categoryRowRef.current;
  const observer = new IntersectionObserver(
    (entries) => {
      const entry = entries[0];
      if (entry) setHeaderDocked(!entry.isIntersecting);
    },
    { rootMargin: `-${HEADER_HEIGHT_PX}px 0px 0px 0px`, threshold: 0 },
  );
  observer.observe(target);
  return () => observer.disconnect();
}, [setHeaderDocked]);
```

**`rootMargin`이 핵심 트릭입니다.** `rootMargin: '-56px 0px 0px
0px'`는 "뷰포트의 위쪽 경계를 56px 안쪽으로 당겨서 관찰한다"는
뜻입니다 — 즉 실제 뷰포트 맨 위가 아니라 **헤더 높이만큼 아래
지점**을 기준선으로 삼습니다. 카테고리 탭(관찰 대상)이 이 당겨진
경계를 지나가는 순간(`isIntersecting`이 `true`→`false`로
바뀌는 순간) 콜백이 호출됩니다. 이러면 "탭이 화면에서 완전히
사라지는 순간"이 아니라 "탭이 고정 헤더 바로 밑에 닿으려는 순간"을
정확히 감지할 수 있습니다.

### "같은 스토어를 보는 두 벌의 사본" 트릭
실제 구현은 `CategoryFilterBar`를 **두 곳에** 렌더링합니다 — 원래
자리(`ProductList` 제목 아래)와 상단 고정 헤더 안. 둘 다 같은
`useCategoryFilterStore`(Zustand)의 `isHeaderDocked` 값을
구독합니다:

```
평소(isHeaderDocked = false):  원래 자리의 탭이 보임, 헤더 쪽 탭은 숨김
도킹 시(isHeaderDocked = true): 원래 자리의 탭은 투명해짐(자리는 유지), 헤더 쪽 탭이 나타남
```

**"원래 자리를 완전히 없애지 않고 투명하게만 만드는" 이유**: 이
위치 자체가 `IntersectionObserver`의 관찰 대상입니다. 만약 도킹되는
순간 이 엘리먼트를 DOM에서 제거하거나 높이를 0으로 만들면, 관찰
대상 자체가 사라지거나 크기가 바뀌면서 스크롤 위치와 감지 결과가
서로 영향을 주는 순환 문제가 생깁니다. `opacity-0
pointer-events-none`(투명 + 클릭 무시)으로만 처리하면 공간은
그대로 차지한 채 눈에만 안 보이고, 클릭 가능한 사본은 항상 실제로
보이는 쪽 하나뿐이라 오작동(숨은 탭을 실수로 클릭)도 없습니다.

**왜 이 상태가 `widgets/product-list` 안이 아니라
`entities/category`에 있는가**: FSD는 "상위 레이어는 하위 레이어만
참조한다"는 단방향 규칙이 있습니다. 이 도킹 상태는 `views/home`
(HomePage, 헤더를 그리는 곳)과 `widgets/product-list`(원래 자리를
그리는 곳) **둘 다**가 읽어야 하는데, `widgets`는 `views`를 참조할
수 없습니다. 두 레이어 모두가 안전하게 참조할 수 있는 **가장 가까운
공통 하위 레이어**가 `entities/category`라서, 상태를 거기로
내렸습니다 — "재사용이 필요하면 더 아래 레이어로 내린다"는 FSD
원칙을 그대로 적용한 사례입니다.

## 서비스워커 캐싱 전략 요약
서비스워커(`app/sw.ts`)는 이 문서에서 다루는 알고리즘 중 유일하게
**두 가지 서로 다른 역할**(캐싱 + 푸시 알림)을 한 파일이 담당합니다.
푸시 알림 부분은 [[07-실시간-알림-아키텍처]]에서 자세히 다뤘으니,
여기서는 캐싱 규칙만 요약합니다.

```ts
const apiNetworkOnly = { matcher: ({ url }) => url.pathname.startsWith('/api/'), handler: new NetworkOnly() };
new Serwist({ runtimeCaching: [apiNetworkOnly, ...defaultCache] });
```

Workbox(Serwist가 내부적으로 쓰는 캐싱 엔진) 라우트는 **배열
순서대로 첫 매치를 사용**합니다. `/api/*` 규칙을 범용
`defaultCache`보다 **앞에** 둬야, 메뉴/재고/주문 같은 실시간 데이터
요청이 정적 리소스용 캐시 우선 전략에 걸려 오래된 데이터를 보여주는
사고를 막을 수 있습니다 — 키오스크는 품절/가격이 실시간으로 바뀌어야
하는 화면이라 이 순서가 중요합니다. 이미지/폰트/CSS/JS 같은 정적
리소스만 `defaultCache`(캐시 우선 계열 전략)를 그대로 씁니다. PWA
설치/매니페스트 자체에 대한 내용은 [[PWA설치]] 참고.

## 전역 이벤트로 만드는 세션 타임아웃(참고)
같은 "전역 상태를 감시해 화면을 바꾼다"는 패턴이 세션 타임아웃
([[세션타임아웃]])에도 등장합니다 — 다만 이건 `IntersectionObserver`가
아니라 `window`의 `pointerdown`/`keydown`/`touchstart`/`wheel`
이벤트를 감시해 유휴 타이머를 계속 재시작시키는, 더 단순한 형태입니다.
모바일 앱에서는 이 감지 방식이 그대로 안 통해서 완전히 다른 메커니즘
(`onStartShouldSetResponderCapture`)을 써야 했는데, 그 이유와 구현은
[[10-모바일-앱-아키텍처]]에서 다룹니다.

## 관련 문서
- [[개발계획서]]
- [[07-실시간-알림-아키텍처]] — 서비스워커의 다른 절반(푸시)
- [[PWA설치]], [[세션타임아웃]], [[카테고리별메뉴조회]]
- [[10-모바일-앱-아키텍처]] — 같은 도킹/타임아웃 개념을 RN에서 어떻게 다르게 풀었는지
