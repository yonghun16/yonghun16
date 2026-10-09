# 균 표시 방식 (현미경 모드 / 캐릭터 모드)

## 🔗 상위 문서
[[03_Germ_and_Clean_Rendering]]

## 관련 노드
[[Settings_and_Persistence]] · [[Dummy_Component_Fallback]] · [[Microscope_Haze_Layer]] · [[Pinch_Zoom_Gesture]] · [[Germ_Generation_Algorithm]]

---

[[Settings_and_Persistence|설정 화면]]에서 고를 수 있는 두 가지 세균 표시 방식.
사실적인 세균 사진을 무서워하는 저학년 아이들을 배려해 나눈 것이며, 값은
`GermDisplayMode = 'MICROSCOPE' | 'CHARACTER'` (`src/types/index.ts`)이다.

| | 현미경 모드 (`MICROSCOPE`) | 캐릭터 모드 (`CHARACTER`, 기본값) |
| :--- | :--- | :--- |
| 세균 그래픽 | `assets/germs/`의 실제 세균 사진(PNG) | `DummyGerm`(표정 있는 캐릭터 6종, Skia 벡터 도형) — [[Dummy_Component_Fallback]] |
| 최대 확대 배율 | **10배** | **4배** — [[Pinch_Zoom_Gesture]] |
| [[Microscope_Haze_Layer\|현미경 뿌연 조명 레이어]] | 있음 | 없음 (사실적 사진이 아니므로 해상도 이질감 자체가 없음) |

## 구현 방식: 새 렌더링 경로를 만들지 않았다
`GermSprite`는 원래부터 "PNG 에셋이 있으면 사진, 없으면(`pngAsset`이 `null`) 벡터
도형"으로 분기하는 [[Dummy_Component_Fallback|이원화 구조]]를 갖고 있었다 (에셋이
아직 준비되지 않은 개발 초기 단계를 위한 설계). `ResultScreen.tsx`는
`germDisplayMode === 'CHARACTER'`일 때 이 분기를 그대로 재사용해서 `pngAsset`에
항상 `null`을 넘긴다 — 캐릭터 모드 전용 컴포넌트를 새로 만들지 않고, 기존 폴백
경로를 "모드"로 승격한 것이다.

```typescript
pngAsset={
  germDisplayMode === 'CHARACTER'
    ? null
    : allGermImages[selectedGermAssetIndices[germ.typeIndex]]
}
```

`maxScale`([[Pinch_Zoom_Gesture]])과 [[Microscope_Haze_Layer|haze 렌더링 여부]]도
같은 `germDisplayMode` 값 하나로 함께 갈린다.
