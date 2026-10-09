---
status: in-progress
created: 2026-09-19
priority: low
tags:
- realtime
- mvp
depends_on:
- 001-room-lifecycle
created_at: 2026-09-19T14:18:10.134348Z
updated_at: 2026-09-20T00:36:28.906900Z
transitions:
- status: planned
  at: 2026-09-20T00:36:18.138608Z
- status: in-progress
  at: 2026-09-20T00:36:18.178999Z
---

# 실시간 채팅

## Overview

대기실/게임 중 언제든 사용 가능한 실시간 채팅. [[게임흐름]] "6. 채팅" 참고.

## Design

- `chat-message`를 broadcast(닉네임 + 메시지 + 타임스탬프)하고, 각 클라이언트는 수신 순서대로 로컬에 누적한다.
- 서버에 영구 저장하지 않으므로, 새로고침 시 채팅 이력도 함께 유실된다. 게임 상태 재동기화(006-reconnect-sync)의 범위에는 채팅 이력 복구를 포함하지 않는다.


입장 후에는(대기실/게임 진행/종료 화면 모두) `RoomLayout`으로 왼쪽 = 단계별 화면, 오른쪽 = `ChatPanel` 2분할로 보여준다(게임흐름.md "UI 요구사항" 참고). 채팅 이력은 방을 나가면(leave) 초기화되지만, "다시 하기"(restart-game)로는 지우지 않는다 — 게임은 새로 시작해도 대화는 이어지는 게 자연스럽다고 판단했다.

## Plan

- [x] 채팅 입력창 + 메시지 리스트 컴포넌트
- [x] `chat-message` broadcast 송수신
- [x] 닉네임/타임스탬프 표시

## Test

- [ ] 대기실/게임 중 어느 시점에도 채팅 송수신이 정상 동작함

(참고) 코드는 구현했지만 실제 Supabase 프로젝트/.env가 없어 여러 탭 간 실시간 송수신은 라이브로 확인하지 못했다(빌드·타입체크·lint만 통과).
