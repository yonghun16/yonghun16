# 세균 애니메이션: 리빌 효과 + 숨쉬기

## 🔗 상위 문서
[[03_Germ_and_Clean_Rendering]]

## 관련 노드
[[Rendering_Architecture]] · [[Pinch_Zoom_Gesture]] · [[Germ_Generation_Algorithm]]

---

세균 하나(`GermSprite.tsx`)에는 서로 다른 두 가지 애니메이션이 동시에 적용된다:
확대 배율에 반응하는 **리빌 효과**와, 시간에 따라 계속 흐르는 **숨쉬기 효과**.

## 1. 리빌(reveal) 효과 — 확대해야 보인다
세균은 확대하지 않은 상태(`scale = 1`)에서는 거의 안 보일 만큼 작고 투명하며,
[[Pinch_Zoom_Gesture|Pinch Zoom]]을 할수록 또렷하고 커진다 ("맨눈으로 안 보이는
세균을 확대해야 발견한다"는 컨셉).

`scale`을 `[MIN_SCALE, maxScale]` 구간에서 아래 두 값으로 `interpolate`한다:
- 불투명도 배율: `[MIN_GERM_OPACITY_FACTOR(0.08), 1]`
- 크기 배율: `[MIN_GERM_ZOOM_FACTOR(0.35), MAX_GERM_ZOOM_FACTOR(1.0)]`

`maxScale`은 [[Germ_Display_Mode|균 표시 방식]]에 따라 10 또는 4이므로, 같은
보간 로직이 두 모드에서 서로 다른 "확대 속도감"으로 자연스럽게 동작한다.

## 2. 숨쉬기(breathing) 애니메이션
세균이 살아있는 것처럼 보이도록, 크기(±8%, `BREATHE_SCALE_AMPLITUDE`)와 불투명도
(±18%, `BREATHE_OPACITY_AMPLITUDE`)가 정현파(sin)로 천천히 오르내린다.

모든 세균이 공유하는 "시계" 값(`breatheClock`, 주기 2400ms, `withRepeat`로 계속
반복)에 [[Germ_Generation_Algorithm|세균마다 다른 위상 오프셋]](`phaseOffset`)을
더해서, 다 같이 숨쉬지 않고 제각각 살아있는 것처럼 보이게 한다.

```typescript
// GermSprite.tsx (개념 요약)
const transform = useDerivedValue(() => {
  const breatheScale = 1 + Math.sin(breatheClock.value + phaseOffset) * BREATHE_SCALE_AMPLITUDE;
  return [{ rotate }, { scale: scale * zoomFactor.value * breatheScale }];
});
```

## ⚠️ Reanimated 클로저 함정
`maxScale`(리빌 효과의 보간 범위 상한)은 `germDisplayMode`라는 일반 prop에서
파생된 일반 JS 값이다. `useDerivedValue`가 이 값을 참조하면서 **명시적 의존성
배열을 넘기지 않으면**, 훅이 처음 만들어질 때의 값에 그대로 고정되어버린다 (AFTER
모드 라이트 스윕 개발 중 실제로 겪은 버그와 같은 종류). `germOpacityFactor` /
`germZoomFactor` / `hazeOpacity` 모두 `[maxScale]` 의존성 배열을 명시한다.
