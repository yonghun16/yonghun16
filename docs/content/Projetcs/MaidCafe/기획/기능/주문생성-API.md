---
title: 주문 생성 API
description: 고객이 제출한 주문을 생성해 저장하는 백엔드 REST API. 요청/응답 필드는 이후 orderType/paymentMethod/orderNumber가 추가되며 확장됨
tags: [기능, 백엔드, 완료]
aliases: []
created: 2026-09-06
updated: 2026-09-10
status: active
---

## 개요
고객 화면에서 제출된 주문을 받아 저장하는 엔드포인트입니다. 작성 당시엔
아이템/총액만 저장했지만, 이후 매장/포장 구분([[매장내포장선택]])과
결제 수단([[결제수단선택]])이 요청에 추가됐고 응답에는 당일 주문번호가
포함되게 됐습니다([[주문완료화면]]에서 사용).

## 동작 방식
`POST /api/orders`가 `packages/types`의 `CreateOrderInput` 타입을 요청
계약으로 사용해, 주문 아이템 목록(상품 id, 이름, 가격, 수량, 옵션
선택값) + 총액 + 매장/포장 구분(`orderType`) + 결제 수단
(`paymentMethod`)을 저장합니다. 응답에는 KST 기준 당일 자정부터 1부터
다시 매기는 짧은 주문번호(`orderNumber`)가 포함되며, 저장 직후(응답을
보낸 뒤) 주문에 담긴 상품의 재고를 자동으로 차감합니다([[재고관리]]
참고). 인증 없이 공개된 엔드포인트입니다(고객이 로그인 없이 바로
주문하므로).

## 관련 코드
- `apps/backend/src/routes/orders.ts` (`POST /api/orders`)
- `packages/types` (`CreateOrderInput`, `Order`)

## 관련 문서
- [[기획서]]
- [[주문제출]] — 이 API를 호출하는 프론트엔드 흐름
- [[매장내포장선택]], [[결제수단선택]], [[주문완료화면]], [[재고관리]]
