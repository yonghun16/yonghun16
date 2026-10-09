# AFTER 모드(손 씻은 후) 연출

## 🔗 상위 문서
[[03_Germ_and_Clean_Rendering]]

## 관련 노드
[[Dummy_Component_Fallback]] · [[Rendering_Architecture]]

---

`washMode === 'AFTER'`일 때 `ResultScreen`에 등장하는, 아이들에게 쾌감을 주기 위한
화려한 연출. 세균 그래픽은 전혀 생성하지 않는다.

## 1. 반짝이(Sparkle) 배치
`src/utils/generateSparkles.ts`가 21개 손 좌표를 기준점 삼아 반짝이를 **18~28개
무작위 개수**, 무작위 각도/반경(±6~24px)으로 흩뿌린다.

> 손가락 마디마다 정확히 하나씩 배치했던 초기 버전은 너무 균일해서 부자연스럽다는
> 피드백에 따라, [[Germ_Generation_Algorithm|세균과 같은 방식]]으로 자연스럽게
> 흩뿌리도록 바꿨다.

## 2. 반짝이 모양: 4방향 별
단순한 원이 아니라, 날카로운 마름모 4개가 십자로 겹친 **4방향 별** 모양이다. 큰 별 +
45° 어긋난 작은 별을 겹쳐서 8방향으로 반짝이는 `✨` 형태를 낸다
(`DummySparkle.tsx`, Skia `Path`로 직접 그림 — [[Dummy_Component_Fallback|반짝이도
PNG 에셋이 없으면 벡터 도형으로 폴백하는 구조]]를 그대로 쓴다. 실제 PNG 에셋은
아직 없어 항상 벡터로 그려진다).

## 3. 트윙클(twinkle) 애니메이션
반짝이마다 다른 위상으로 크기(0.8~1.15배)와 불투명도가 정현파로 깜빡인다
(`twinkleClock`, 주기 1600ms, `withRepeat`).

## 4. 화면 전체 플래시
결과 화면 진입 시, 화면 중앙에서 원형으로 확 밝아졌다가 퍼지며 사라지는 연출
(`Circle` + `RadialGradient`, 흰색, 900ms). "눈부신 광원을 보는 것처럼" 화면 전체가
반짝이는 효과를 원한다는 피드백에 따라 도입했다.

> **이전 시도**: 손 윤곽(convex hull)만 마스킹해서 빛줄기가 손바닥 위로 훑고
> 지나가는 라이트 스윕 연출이었다. 구현 중 두 가지 버그를 겪었다 — (1)
> `useDerivedValue`가 일반 JS 값을 의존성 배열 없이 참조해서 레이아웃 계산 전
> 값에 고정되는 문제, (2) `LinearGradient`의 좌표가 고정돼 있어서 움직이는
> `Rect`가 그 범위를 벗어나면 빛줄기가 끊긴 것처럼 보이는 문제. 두 버그를 고친
> 뒤에도 "화면 전체를 훑고 지나가서 손바닥만 빛나는 느낌이 안 난다"는 피드백이
> 이어져, 결국 손 마스크 접근 자체를 버리고 화면 전체 플래시로 교체했다. 당시
> 관련 유틸(`convexHull.ts`, 볼록 껍질 기반 손 윤곽)은 삭제했었지만, 손 모양 안에서만
> 무작위 배치가 필요한 다른 용도([[Hand_Silhouette_Sampling|세균 위치 샘플링]])가
> 생기면서 더 정교한 형태(손가락 캡슐 + 손바닥 다각형, `geometry.ts` +
> `handSilhouette.ts`)로 다시 만들어졌다. 지금의 `flashOpacity`/`flashRadius`는
> (1)의 교훈을 반영해 `containerSize`에 대한 의존성 배열을 명시하고, (2)의 교훈을
> 반영해 `Circle`과 `RadialGradient`가 같은 `flashRadius` 공유값을 쓴다.

## 5. 칭찬 메시지 + 피드백
상단에 "✨ 참 잘했어요! 세균이 깨끗하게 사라졌어요!" 팝업, 진동
(`Haptics.NotificationFeedbackType.Success`), `playCleanSound()`(현재 no-op).
