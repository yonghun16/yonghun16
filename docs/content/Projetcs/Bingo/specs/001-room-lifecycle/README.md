---
status: in-progress
created: 2026-09-19
tags:
- realtime
- mvp
created_at: 2026-09-19T14:17:47.863832Z
updated_at: 2026-09-30T05:50:54.351118Z
transitions:
- status: planned
  at: 2026-09-19T14:57:51.465849Z
- status: in-progress
  at: 2026-09-19T14:57:51.510164Z
---

# 방 생성/입장/정원/호스트 판단

## Overview

카카오톡 링크 공유로 방을 만들고 입장하는 흐름. Supabase Realtime 채널(`bingo-room-{roomId}`) 구독과 Presence로 참가자/정원/호스트를 관리한다. 전체 배경은 [[게임흐름]] 참고.

## Design

- 방 생성 시 클라이언트가 roomId를 직접 생성하고 `/room/:roomId`로 라우팅한다.
- 입장 시 채널을 구독하기 전에 현재 Presence 상태를 조회해 정원(5명) 초과 여부를 확인하고, 초과 시 입장을 막는다.
- 이미 게임이 시작된 방(Presence로 공유되는 status가 `playing`/`ended`)에는 입장할 수 없다.
- 호스트 = Presence 입장 시각(`joinedAt`)이 가장 이른 참가자. 별도 broadcast 없이 모든 클라이언트가 동일한 Presence 목록으로 각자 동일하게 계산한다.
- 별도 게임 서버 없이 Presence/Broadcast만 사용하는 배경은 `docs/decisions/0001-supabase-realtime-통신.md`(ADR) 참고.


- 닉네임은 Presence key로 사용되므로 방 안에서 유일해야 한다. 입장 시 현재 Presence 목록에 동일 닉네임이 있으면 입장을 막고 다른 닉네임을 요구한다(같은 사람의 다중 탭 재입장도 동일하게 처리).
- 참가자가 0명인 방에 입장하는 것은 정상 동작이며, 입장한 사람이 대기 상태부터 새로 시작하는 것과 같다(서버에 남은 이전 기록 없음).


방 만들기 시 보드 크기(3x3/4x4/5x5/6x6/7x7, 기본 5x5)도 함께 고른다. 서버가 없어 Realtime으로 동기화할 수 없으므로, 초대 링크의 `?size=` 쿼리 파라미터로 고정해 전달한다 — 그 링크로 들어오는 모든 참가자가 URL만으로 동일한 크기를 쓰게 된다.

## Plan

- [x] `lib/supabase.ts`: Supabase 클라이언트 단일 인스턴스
- [x] `features/bingo-room/api/roomChannel.ts`: 채널 구독/구독 해제
- [x] `useRoomPresence` 훅: 참가자 목록, 인원수, 호스트 여부 계산
- [x] 정원(5명) 초과 / 이미 시작된 방 입장 차단 로직
- [x] 방 만들기 → roomId 생성 → 초대 링크 표시 UI
- [x] 닉네임 입력 → 입장 플로우


- [x] 입장 시 Presence 목록에서 닉네임 중복 체크, 중복 시 입장 차단 + 에러 메시지

## Test

- [ ] 여러 탭으로 방 생성/입장 시 Presence로 참가자 목록이 실시간 동기화됨
- [ ] 5명 입장 후 6번째 입장 시도 시 "방이 가득 찼습니다" 안내가 뜸
- [ ] 이미 게임이 시작된 방에 새로 입장 시도하면 차단됨
- [ ] 모든 탭에서 동일한 사람이 호스트로 계산됨


- [ ] 이미 사용 중인 닉네임으로 입장 시도 시 차단되고 에러 메시지가 표시됨
- [ ] 참가자가 0명인 방에 입장하면 정상적으로 대기 상태부터 시작됨

## Notes

호스트 이탈 시 승계 로직의 실제 활용(턴 타이머 이어받기)은 003(turn-gameplay)에서 다룬다.

"정원(5명) 초과 / 이미 시작된 방 입장 차단"은 002-board-setup에서 각자 Presence에 roomStatus를 함께 실어보내는 방식으로 마무리했다(각 참가자가 game-started 수신 시 자신의 presence를 roomStatus: 'playing'으로 다시 track). 늦게 입장하는 클라이언트도 기존 참가자 중 한 명의 presence만 보면 시작 여부를 알 수 있다.
Test 항목은 아직 실제 Supabase 프로젝트/.env가 없어 브라우저로 라이브 검증하지 못했다(빌드·타입체크·lint만 통과 확인). .env.example을 채운 뒤 여러 탭으로 직접 확인이 필요하다.