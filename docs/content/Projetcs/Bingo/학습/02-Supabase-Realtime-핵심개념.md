---
title: Supabase Realtime 핵심 개념
tags: [튜토리얼]
description: Channel / Presence / Broadcast가 뭔지, 이 프로젝트가 서버 없이 실시간 동기화를 만든 방법
created: 2026-09-20
updated: 2026-09-30
status: active
---

이 프로젝트는 별도 게임 서버 없이 Supabase Realtime만으로 5명이 실시간으로 같이 게임하는 걸 구현했습니다. 그 뼈대가 되는 세 개념(Client, Presence, Broadcast)을 실제 코드로 설명합니다.

## 1. 클라이언트 인스턴스는 하나만 (`lib/supabase.ts`)

```ts
// src/lib/supabase.ts
let client: SupabaseClient | null = null;

export function getSupabaseClient(): SupabaseClient {
  if (!client) {
    const env = getEnv();
    client = createClient(env.supabaseUrl, env.supabaseAnonKey);
  }
  return client;
}
```

`createClient()`를 여기저기서 호출하면 안 됩니다 — 하나의 앱에는 하나의 클라이언트 인스턴스만 있어야 커넥션 관리가 꼬이지 않습니다. 그래서 "누구든 Supabase가 필요하면 이 함수를 호출해서 캐시된 인스턴스를 받아간다"는 지연 초기화(lazy singleton) 패턴을 씁니다.

**왜 즉시 생성(`export const supabase = createClient(...)`)이 아니라 함수로 감쌌는가?** 즉시 생성하면 이 파일을 import하는 순간 환경변수(`.env`)가 없으면 바로 에러가 납니다. `RoomPage`가 이 체인을 정적으로 import하고 있어서, `.env` 없이는 **Supabase와 아무 상관 없는 랜딩 페이지까지 하얀 화면**이 되는 실제 버그가 있었습니다. 함수로 감싸면 "진짜로 Supabase를 쓰는 순간"(방에 입장할 때)에만 초기화가 일어나서 이 문제가 사라집니다.

## 2. 채널(Channel) — 방 하나 = 채널 하나

```ts
// src/features/bingo-room/api/roomChannel.ts
function roomChannelName(roomId: string): string {
  return `bingo-room-${roomId}`;
}

export function createRoomChannel(roomId: string, nickname: string): RealtimeChannel {
  return getSupabaseClient().channel(roomChannelName(roomId), {
    config: { presence: { key: nickname } },
  });
}
```

`supabase.channel(이름, 옵션)`으로 채널을 "만들기"만 하고, 여기서는 아직 `.subscribe()`를 부르지 않습니다 — 이유는 바로 아래 5번에서 다룹니다(실제로 겪은 버그입니다). **채널 이름이 같은 클라이언트끼리만 서로 통신합니다** — 그래서 방마다 다른 이름(`bingo-room-{roomId}`)을 쓰면, 방 A의 이벤트가 방 B로 새는 일이 없습니다.

`config.presence.key`는 "이 채널 안에서 나를 어떤 이름으로 식별할지"입니다. 이 프로젝트는 닉네임을 그대로 key로 씁니다 — 그래서 "닉네임은 방 안에서 유일해야 한다"는 규칙이 자연스럽게 생깁니다(001 스펙).

## 3. Presence — "지금 누가 접속해 있는가"를 자동으로 동기화

```ts
export function trackPresence(channel: RealtimeChannel, payload: RoomPresencePayload) {
  return channel.track(payload);
}
```

`channel.track(내_정보)`를 부르면, 채널을 구독 중인 **모든 클라이언트**가 "누군가 들어왔다"는 걸 자동으로 알게 됩니다. 반대로 `untrack()`을 부르거나 브라우저 탭이 닫히면(연결이 끊기면) 자동으로 "누군가 나갔다"는 게 전파됩니다 — **이걸 우리가 직접 구현할 필요가 없습니다.** Presence는 "지금 이 순간 채널에 붙어있는 사람 목록"을 관리해주는 기능이라고 생각하면 됩니다.

```ts
channel.on("presence", { event: "sync" }, () => {
  setPlayers(toPlayers(channel.presenceState<RoomPresencePayload>()));
});
```

`presenceState()`는 "지금 이 채널에 track되어 있는 모든 사람의 정보"를 객체로 돌려줍니다. `sync` 이벤트는 이 목록이 바뀔 때마다(누가 들어오거나 나갈 때) 발생합니다. 이 프로젝트는 참가자 목록, 방장이 누군지, 준비 상태, 완성 라인 수까지 전부 이 Presence 하나로 동기화합니다 — 이유는 [[04-핵심-설계-결정]]에서 다룹니다.

**중요한 특성**: Presence는 "지금 접속 중인 사람의 최신 상태"만 보여줍니다. 과거 기록이 아닙니다. 그래서 새로 들어온 사람도 `presenceState()`를 한 번 읽으면 "지금 누가 있고 각자 상태가 뭔지"를 바로 알 수 있습니다(001 스펙의 닉네임 중복/정원 체크, 006 스펙의 "이미 시작된 방인지" 판단이 전부 이 성질을 이용합니다).

## 4. Broadcast — "방금 일어난 일"을 전체에 알리기

```ts
export function broadcast<T extends object>(channel: RealtimeChannel, event: string, payload: T) {
  return channel.send({ type: "broadcast", event, payload });
}
```

Presence가 "지금 상태"라면, Broadcast는 "방금 벌어진 사건"입니다. 예를 들어 "13번이 호출됐다"(`number-called`), "누가 채팅을 보냈다"(`chat-message`) 같은 것들이죠.

```ts
channel.on("broadcast", { event: "number-called" }, ({ payload }) => {
  applyNumberCalled(payload as NumberCalledPayload);
});
```

`.on("broadcast", { event: "이벤트이름" }, 콜백)`으로 특정 이벤트를 구독합니다. 이 프로젝트에서 쓰는 이벤트 목록은 `src/features/bingo-room/types/events.ts`에 타입으로 다 정의돼 있고, 왜 이런 이벤트들이 필요한지는 [[게임흐름]] 문서의 "Supabase Realtime 이벤트" 섹션에 정리돼 있습니다.

**아주 중요한 특성 — self-echo가 기본적으로 꺼져 있다**: 내가 `broadcast()`로 보낸 이벤트는 **나 자신에게는 돌아오지 않습니다.** 그래서 이 코드 전체에 이런 패턴이 반복됩니다.

```ts
void broadcast(channel, "number-called", payload);
applyNumberCalled(payload);   // 내가 보낸 것도 내가 직접 한 번 더 적용해준다
```

"보내기"와 "내 화면에 반영하기"를 항상 같이 해줘야 한다는 뜻입니다. 이걸 깜빡하면 "정작 이벤트를 보낸 사람의 화면만 안 바뀌는" 버그가 생깁니다.

## 5. `subscribe()`는 모든 리스너를 등록한 뒤 딱 한 번만 — 실제로 겪은 버그

Supabase Realtime은 **`channel.subscribe()`를 호출한 뒤에는 `channel.on(...)`으로 리스너를 더 추가할 수 없습니다**("cannot add ... callbacks after subscribe()" 에러). 처음엔 `createRoomChannel`(당시 이름은 `subscribeToRoomChannel`) 안에서 채널을 만들자마자 바로 `.subscribe()`를 불렀는데, 그 뒤에 `join()`이 `channel.on("presence", ...)` 같은 리스너를 추가하려다 이 에러로 막혔습니다.

해결책은 **"채널 만들기"와 "구독 시작하기"를 분리**하는 것입니다. `createRoomChannel`은 채널 객체만 만들어 돌려주고, 호출하는 쪽(`join()`)이 필요한 `.on()` 리스너를 전부 등록한 다음 마지막에 딱 한 번 `.subscribe(콜백)`을 부릅니다.

```ts
// useRoomPresence.ts의 join() 안
const channel = createRoomChannel(roomId, nickname);

channel.on("presence", { event: "sync" }, () => { /* ... */ });
channel.on("broadcast", { event: "game-started" }, () => { /* ... */ });
// ... 필요한 리스너를 전부 먼저 등록 ...

channel.subscribe((status) => {
  if (status !== "SUBSCRIBED") return;
  const state = channel.presenceState<RoomPresencePayload>();
  // 정원 체크, 닉네임 중복 체크 ...
});
```

채널 구독은 즉시 완료되지 않고 네트워크를 타는 비동기 작업이기도 합니다. `SUBSCRIBED` 상태가 되기 전에 `presenceState()`를 읽으면 아직 다른 사람들의 정보가 도착하지 않았을 수 있습니다. 그래서 "완전히 연결된 뒤"에만 정원/중복 체크 같은 판단을 합니다.

**배울 점**: 라이브러리가 "한 번 호출하면 이후 다른 호출을 막는" 제약을 두는 경우가 있습니다. 이런 제약은 문서를 안 읽고 에러 메시지만 보고도 원인을 찾을 수 있을 만큼 에러 문구가 친절한 편이니, 당황하지 말고 에러 메시지부터 정독하는 습관이 도움이 됩니다.

## 관련 문서
- [[00-학습-가이드]]
- [[01-리액트-패턴]]
- [[04-핵심-설계-결정]]
