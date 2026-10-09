# Pinch Zoom + Pan 제스처

## 🔗 상위 문서
[[03_Germ_and_Clean_Rendering]]

## 관련 노드
[[Rendering_Architecture]] · [[Germ_Display_Mode]] · [[Germ_Animations]]

---

- `react-native-gesture-handler`의 `Gesture.Pinch()` + `Gesture.Pan()`을
  `Gesture.Simultaneous()`로 결합해서 사용한다.
- 사진과 세균/반짝이 레이어를 하나의 Skia `Group transform`으로 묶어 **1:1 동시
  확대/축소**가 이뤄지도록 구현한다 ([[Rendering_Architecture]] 참고).
- Pan은 확대 상태(`scale > 1`)에서만 동작하며, 이동 범위는 확대 배율에 따라
  클램프된다 (`clampTranslation`).

## 최대 확대율은 [[Germ_Display_Mode|균 표시 방식]]에 따라 다르다

| 모드 | 최대 배율 | 비고 |
| :--- | :--- | :--- |
| 현미경 모드 | **10.0x** | 최초 기획안은 4.0x였지만, "진짜 현미경처럼 손 피부 표면까지 파고들어 보이도록" 상향 |
| 캐릭터 모드 | **4.0x** | 최초 기획안 그대로 유지 |

```typescript
const maxScale = germDisplayMode === 'CHARACTER' ? CHARACTER_MAX_SCALE : MICROSCOPE_MAX_SCALE;
// ...
scale.value = clamp(savedScale.value * event.scale, MIN_SCALE, maxScale);
```

`maxScale`은 [[Germ_Animations|세균 리빌 효과의 보간 범위]]에도 그대로 쓰인다.
