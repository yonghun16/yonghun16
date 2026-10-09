---
title: 주문 상태 전이 API
description: 주문은 진행중/완료/취소 3단계로 전이됨(완료↔진행중, 진행중→취소는 되돌릴 수 있음). 접수→준비중→완료→픽업 같은 세분화는 검토 후 필요 없다고 결정
tags: [기능, 백엔드, 완료]
aliases: []
created: 2026-09-06
updated: 2026-09-13
status: active
---

## 개요
주문은 생성 직후 `isCompleted: false, isCancelled: false`(진행중)
상태로 시작합니다. 관리자가 "완료" 처리하면 `isCompleted: true`(완료)로
바뀌고, 필요하면 "되돌리기"로 다시 진행중으로 되돌릴 수 있습니다.
진행중 주문은 "취소"로 `isCancelled: true`(취소됨)로 바꿀 수 있습니다.

접수/준비중/완료/픽업처럼 더 세분화된 **준비 단계**가 필요한지는
로드맵에 "검토 필요" 항목으로 남아있었는데, "주방에서 실제로 더
세분화된 단계가 필요한가"라는 질문에 "지금의 진행중/완료 2단계로
충분하다"고 답해 **세분화하지 않기로 결정**했습니다 — 작은 매장/단일
카운터 운영 규모에서는 "진행중 주문" 탭에 뜬 주문을 준비해서 "완료"
버튼 한 번 누르는 흐름만으로 충분하다는 판단입니다.

**취소는 이 결정과 별개의 축입니다.** 위 결정은 "정상적으로 진행되는
주문을 얼마나 잘게 쪼갤지"에 대한 것이고, 취소는 "애초에 완료되지
못하고 끝나는 주문을 어떻게 기록할지"에 대한 것입니다. 실수로 잘못
받은 주문이나 손님의 취소 요청에 대응할 방법이 없다는 게 부수적으로
발견되어, 완료 처리를 잘못 눌렀을 때 되돌리는 기능과 함께 추가했습니다.

## 동작 방식
- `PATCH /api/orders/:id/complete` — 진행중 주문을 완료 처리합니다.
- `PATCH /api/orders/:id/uncomplete` — 완료 처리를 취소하고 다시
  진행중으로 되돌립니다. 이미 진행중인 주문에는 400을 반환합니다.
- `PATCH /api/orders/:id/cancel` — **완료되지 않은(진행중) 주문만**
  취소합니다. 이미 완료됐거나 이미 취소된 주문은 400을 반환합니다
  (완료된 주문을 취소하려면 먼저 되돌리기로 진행중으로 만들어야
  합니다). 취소 시 주문 생성 때 차감했던 재고를 되돌립니다
  (`restoreStockForOrder`, [[재고관리]] 참고) — 재고 복원은 취소 자체의
  성공/실패와 분리해 별도 `try/catch`로 감싸므로, 복원이 실패해도
  취소 처리 자체는 이미 반영됩니다([[05-주문-핵심-알고리즘]]의
  fire-and-forget 패턴과 같은 이유).
- 세 라우트 모두 `requireAdmin`으로 보호됩니다.
- `GET /api/orders`는 `status=pending|completed|cancelled` 쿼리로 세
  상태를 나눠 조회할 수 있습니다. `pending`/`completed`는 취소된
  주문을 제외하고, `cancelled`는 취소된 주문만 반환합니다.
  [[주문내역조회]]의 "진행중 주문"/"완료한 주문"/"취소된 주문" 3개 탭이
  이 API를 사용합니다.
- 취소된 주문은 완료로 전이될 수 없는 종결 상태입니다 — 상태 전이는
  `진행중 ⇄ 완료`, `진행중 → 취소`(단방향)만 존재합니다.

## 관련 코드
- `apps/backend/src/models/Order.ts` (`isCompleted`/`isCancelled`
  필드, 둘 다 기본값 `false`)
- `apps/backend/src/routes/orders.ts` — `PATCH /api/orders/:id/complete`,
  `PATCH /api/orders/:id/uncomplete`, `PATCH /api/orders/:id/cancel`,
  `GET /api/orders`의 `status` 쿼리 필터링
- `apps/backend/src/lib/inventory.ts` — `restoreStockForOrder`(취소 시
  재고 복원, `decrementStockForOrder`의 역연산)
- `apps/backend/tests/orders.test.ts` — 취소/되돌리기/재고 복원/통계
  제외 테스트
- `packages/types`의 `Order`(`isCompleted`/`isCancelled` 필드),
  `OrderStatusFilter`(`'pending' | 'completed' | 'cancelled'`)
- `apps/frontend/src/entities/order/api/orderApi.ts` (`uncompleteOrder`,
  `cancelOrder`)
- `apps/frontend/src/entities/order/ui/OrderCard.tsx` (진행중 주문의
  "취소"/"완료" 버튼, 완료한 주문의 "되돌리기" 버튼, 취소된 주문의
  "❌ 취소됨" 배지)

## 범위 밖
- 접수/준비중/완료/픽업 같은 세분화된 상태 전이 — 위 결정에 따라
  범위 밖으로 정리. 매장 규모가 커지거나 운영 방식이 바뀌면 그때
  다시 검토합니다.
- 완료된 주문의 직접 취소 — 되돌리기를 먼저 거치게 강제해, "이미
  손님에게 전달한 주문"과 "아직 안 나간 주문"을 상태만으로 실수 없이
  구분할 수 있게 했습니다.

## 관련 문서
- [[기획서]]
- [[주문내역조회]]
- [[재고관리]] — 취소 시 재고를 되돌리는 `restoreStockForOrder`
- [[매출통계집계-API]] — 취소된 주문을 통계 집계에서 제외하는 이유
