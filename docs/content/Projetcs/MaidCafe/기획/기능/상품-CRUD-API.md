---
title: 상품 CRUD API
description: 상품(메뉴) 조회/등록/수정/품절처리/재고/순서변경/삭제를 처리하는 백엔드 REST API 엔드포인트 전체 목록. 각 엔드포인트의 실제 동작은 개별 기능 문서 참고
tags: [기능, 백엔드, 완료]
aliases: []
created: 2026-09-06
updated: 2026-09-10
status: active
---

## 개요
프론트엔드(고객 화면, 관리자 화면)가 사용하는 상품 관련 REST
엔드포인트입니다. 작성 당시엔 조회/등록/삭제 3개뿐이었지만, 이후
수정·품절 처리·재고·순서변경 엔드포인트가 추가되면서 각 기능별
문서([[메뉴등록]], [[메뉴삭제]], [[메뉴수정품절처리]], [[재고관리]])가
더 상세하고 최신인 설명을 갖게 됐습니다 — 이 문서는 전체 엔드포인트
목록만 정리하고, 각각의 실제 동작/화면/히스토리는 해당 문서를
참고하세요(같은 내용을 여러 문서에 중복 서술하지 않기 위함).

## 동작 방식
Express 5 + Mongoose로 구현되어 있으며(`apps/backend/src/routes/products.ts`),
`GET /api/products`(목록 조회, 공개), `POST /api/products`(등록,
[[메뉴등록]]), `PUT /api/products/:id`(수정), `PATCH
/api/products/:id/sold-out`(품절 토글), `PATCH /api/products/:id/stock`
(재고 절대값 설정, [[재고관리]]), `PATCH /api/products/reorder`(같은
카테고리 안 순서 변경), `DELETE /api/products/:id`(삭제,
[[메뉴삭제]])를 제공합니다 — 조회를 뺀 나머지는 전부 `requireAdmin`으로
보호됩니다([[관리자인증인가]] 참고). 입력 검증은 Mongoose 스키마의
`required`/`enum`에 의존합니다 ([[기획서]]의 "알려진 제약" 참고).

## 관련 코드
- `apps/backend/src/routes/products.ts`

## 관련 문서
- [[기획서]]
- [[메뉴등록]], [[메뉴삭제]], [[메뉴수정품절처리]], [[재고관리]] — 각
  엔드포인트의 실제 화면/동작/히스토리
- [[관리자인증인가]] — 조회를 뺀 엔드포인트를 보호하는 인증 메커니즘
