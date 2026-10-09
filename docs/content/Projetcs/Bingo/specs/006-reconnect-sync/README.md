---
status: in-progress
created: 2026-09-19
tags:
- realtime
depends_on:
- 001-room-lifecycle
- 003-turn-gameplay
created_at: 2026-09-19T14:18:10.175784Z
updated_at: 2026-09-20T02:44:55.312469Z
transitions:
- status: planned
  at: 2026-09-20T01:02:15.356855Z
- status: in-progress
  at: 2026-09-20T01:02:15.405878Z
---

# 재접속/새로고침 상태 동기화

## Overview

새로고침이나 재접속으로 로컬 상태를 잃은 클라이언트가 현재 게임 상태를 다시 받아오는 기능. [[게임흐름]] "예외 처리" 및 `docs/decisions/0001-supabase-realtime-통신.md`(ADR)의 Consequences 참고.

## Design

- 재접속한 클라이언트는 채널 재구독 직후 `request-state`를 broadcast한다.
- 다른 클라이언트(주로 호스트, 없으면 아무나)가 자신이 알고 있는 최신 상태를 `state-sync`로 응답한다.
- 알려진 한계: 참가자 전원이 동시에 새로고침하면 상태를 복구해 줄 클라이언트가 없다. 이는 서버가 상태 원본을 갖지 않는 구조(`docs/decisions/0001-supabase-realtime-통신.md`)의 트레이드오프다. 이 스펙에서는 완전한 복구 대신, 일정 시간(약 3초) 무응답 시 안내 메시지와 함께 방을 새로 시작하도록 유도하는 것으로 대응한다.


구현하면서 두 가지를 바꿨다.
- "호스트 우선 응답" 대신, 그 시점에 `roomStatus`가 `waiting`이 아닌 클라이언트라면 누구든 `state-sync`로 응답하게 단순화했다. 응답 내용은 다들 같은 broadcast 이력에서 계산한 것이라 서로 다르지 않으므로, 먼저 온 것만 채택하고 중복 응답은 그냥 버린다.
- 재접속 대상이 "진짜 원래 참가자"인지 "방금 시작된 게임에 끼어들려는 낯선 사람"인지 구분하는 문제가 있었다 — `state-sync`의 `turnOrder`(게임 시작 시점에 고정된 참가자 id 목록)에 닉네임이 있는지로 판별한다. 없으면 "이미 시작된 방"과 동일하게 거부한다.
- (미리 필요했던 인프라) 닉네임과 내 보드는 `localStorage`에도 저장한다. 방 상태는 재접속으로 복구되지만, 내 보드는 애초에 아무에게도 broadcast된 적이 없어서(비공개 정책) 같은 브라우저 로컬 저장소 말고는 복구할 방법이 없다.

## Plan

- [x] 재구독 시 `request-state` broadcast
- [x] `state-sync` 응답 로직(호스트 우선 응답, 없으면 아무나)
- [x] 수신한 상태로 로컬 Room/Player 상태 재구성


- [x] `request-state` 응답 대기 타임아웃(약 3초) 처리 + 무응답 시 안내 UI

## Test

- [ ] 게임 진행 중 새로고침한 탭이 몇 초 내에 현재 상태로 복구됨
- [ ] 세팅 단계에서 새로고침해도 참가자 목록/준비 상태가 정상 복구됨

- [ ] `request-state`에 3초 내 응답이 없으면 안내 메시지가 뜨고 방을 새로 시작할 수 있음

(참고) 코드는 구현했지만 실제 Supabase 프로젝트/.env가 없어 여러 탭으로 하는 라이브 검증(재접속, 3초 타임아웃, turnOrder 기반 거부 등)은 하지 못했다(빌드·타입체크·lint만 통과).
