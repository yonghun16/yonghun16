---
status: in-progress
created: 2026-09-19
tags:
- realtime
- mvp
- gameplay
depends_on:
- 003-turn-gameplay
created_at: 2026-09-19T14:18:03.900378Z
updated_at: 2026-09-19T23:50:58.074651Z
transitions:
- status: planned
  at: 2026-09-19T23:50:40.107951Z
- status: in-progress
  at: 2026-09-19T23:50:40.150544Z
---

# 게임 종료 및 재시작

## Overview

우승자를 확정하고 게임을 종료·재시작하는 흐름. [[게임흐름]] "5. 게임 종료" 참고.

## Design

- 호스트는 같은 `number-called` 라운드에서 들어오는 `bingo-completed` 이벤트를 짧은 유예 시간(약 300ms) 동안 모아 동시 달성 여부를 판단한다.
- 확정된 우승자(단독 또는 복수)와 전체 참가자 보드 공개 데이터를 `game-over`로 broadcast한다. 이 즉시 게임이 종료된다.
- "다시 하기" 버튼을 누르면 `restart-game`을 broadcast하고, 수신한 전원이 로컬 상태를 초기화한 뒤 세팅 단계(002-board-setup)로 복귀한다.


- `bingo-completed` 동일 라운드 판단은 `turnSeq`로 한다 — 유예 시간(약 300ms) 동안 모은 `bingo-completed` 중 같은 `turnSeq`를 가진 것들만 동시 달성으로 묶어 공동 우승 처리한다.
- 게임 진행 중 이탈로 활성 참가자가 1명만 남으면, 유예 시간을 기다리지 않고 즉시 그 1명을 단독 우승자로 하는 `game-over`를 broadcast한다([[게임흐름]] "예외 처리 > 게임 진행 중 이탈" 참고, 003-turn-gameplay에서 감지).


구현하면서 `game-over` payload를 바꿨다: 호스트는 다른 참가자의 보드 내용을 갖고 있지 않으므로(001~003 내내 보드는 각자 로컬에만 있음) `winnerIds`(우승자 id 목록)만 담아 보낸다. "전체 보드 공개"는 `game-over`를 받은 각 클라이언트가 곧바로 자신의 보드를 `board-revealed`로 broadcast하는 방식으로 구현했다(게임흐름.md 동기화 완료).

## Plan

- [x] 호스트: `bingo-completed` 유예 수집 + 동시 우승 판정 로직
- [x] `game-over` broadcast(우승자 정보 + 전체 보드 공개 데이터)
- [x] 우승자 배너/모달 UI
- [x] 전체 참가자 보드 공개 뷰
- [x] "다시 하기" 버튼 → `restart-game` broadcast
- [x] `restart-game` 수신 시 로컬 상태 초기화


- [x] `bingo-completed`를 `turnSeq` 기준으로 그룹핑해 동시 달성 판정
- [x] 활성 참가자 1명 남음 감지 시 유예 없이 즉시 `game-over` 발행

## Test

- [ ] 단독 우승 시 정확히 한 명만 우승자로 표시됨
- [ ] 동시 달성(같은 호출로 여러 명이 3줄 완성) 시 공동 우승으로 표시됨
- [ ] 게임 종료 즉시 모든 참가자의 보드가 공개됨(마스킹 해제)
- [ ] 재시작 시 이전 라운드 데이터가 모든 탭에서 남지 않고 완전히 초기화됨

- [ ] 서로 다른 turnSeq에서 들어온 완성 이벤트는 같은 라운드로 묶이지 않음
- [ ] 이탈로 활성 참가자가 1명이 되면 유예 시간 없이 즉시 게임이 종료되고 그 1명이 단독 우승자로 표시됨

(참고) 위 항목들은 코드로 구현했지만 실제 Supabase 프로젝트/.env가 없어 브라우저 라이브 검증(동시 우승, 1명 생존 자동 종료, 재시작 후 상태 초기화 등)은 하지 못했다(빌드·타입체크·lint만 통과).
