# 렌더링 아키텍처: Skia 내부 transform으로 확대하는 이유

## 🔗 상위 문서
[[03_Germ_and_Clean_Rendering]]

## 관련 노드
[[Pinch_Zoom_Gesture]] · [[Germ_Animations]]

---

사진, 세균, 반짝이를 모두 `useImage`/Skia 도형으로 `Canvas` 안에 그리고, Pinch Zoom도
RN `View`의 CSS `transform: scale`이 아니라 **Skia `Group`의 `transform` prop**으로
처리한다 (`ResultScreen.tsx`).

## 문제: CSS transform 방식의 흐림
CSS transform 방식은 이미 작게 래스터화된 결과물을 그대로 늘리기만 해서 확대할수록
흐려지는 문제가 실기기에서 확인됐다. 세균 이미지의 원본 해상도를 키워봐도(500px → 더
큰 파일) 해결되지 않았는데, 원인이 절대 해상도가 아니라 **"이미 작게 그려진 비트맵을
늘리는" 아키텍처 자체**였기 때문이다 (스트레치 비율에 의한 블러는 원본 픽셀 수와
무관하다).

## 해결: Skia 내부 transform
Skia 내부 transform은 매 프레임 원본 해상도에서 다시 그리므로, 몇 배를 확대해도
(사진 자체의 해상도 한계를 빼면) 선명하다. `contentTransform`(`useDerivedValue`)을
사진 + 세균 + 반짝이를 감싸는 하나의 `Group`에 적용해서 **1:1 동시 확대/축소**가
이뤄지도록 구현한다.

이 아키텍처 전환 덕분에 [[Pinch_Zoom_Gesture|최대 확대 배율을 4배 → 10배로]] 올려도
세균이 흐려지지 않게 됐고, 다만 사진(피부) 쪽 해상도 한계는 여전히 남아있어
[[Microscope_Haze_Layer|현미경 뿌연 조명 레이어]]로 그 이질감을 가린다.
