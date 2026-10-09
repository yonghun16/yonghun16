# 세균 생성 알고리즘

## 🔗 상위 문서
[[03_Germ_and_Clean_Rendering]]

## 관련 노드
[[Germ_Display_Mode]] · [[Germ_Animations]] · [[Dummy_Component_Fallback]] · [[Hand_Silhouette_Sampling]]

---

`src/utils/generateGerms.ts`가 21개 손 좌표를 기준으로 세균 객체(`GermObject[]`)를
무작위 생성하고, `GermSprite.tsx`가 각 객체를 Skia로 그린다. `washMode === 'BEFORE'`
일 때만 생성된다.

> ⚠️ **좌표 변환 필수 규칙**: MediaPipe 좌표는 `0.0~1.0` 정규화 값이므로, 화면에
> 그릴 때는 `rect.x + x * rect.width`, `rect.y + y * rect.height`로 변환한다.
> `rect`는 사진이 실제로 표시되는 영역(letterbox 보정 포함, `computeContainRect()`).

## 생성 규칙
1. **개수**: 55 ~ 90개 무작위 (`MIN_GERM_COUNT` / `MAX_GERM_COUNT`). 최초 기획안은
   15~30개였으나, 더 북적이게 해달라는 피드백에 따라 여러 차례 늘렸다 (15~30 →
   25~45 → 35~60 → 55~90).
2. **위치**: 손 실루엣(손가락 캡슐 + 손바닥 다각형) 위에 균일 분포로 무작위
   배치한다 → [[Hand_Silhouette_Sampling]].
3. **회전/스케일/불투명도**: 무작위 회전각(0~360°), 무작위 스케일(0.8~1.3), 무작위
   불투명도(0.4~0.7).
4. **종류** (`typeIndex`, 0~5): 아래 참고.
5. **애니메이션 위상** (`phaseOffset`, 0~2π): [[Germ_Animations|숨쉬기 애니메이션]] 위상 오프셋.

## 세균 종류 무작위 선택 (현미경 모드 전용)
`assets/germs/`에는 실제 세균 사진 10종이 있다. 한 손에 10종을 다 쓰면 산만하므로
`ResultScreen`이 사진을 찍을 때마다(= `photoUri`가 바뀔 때마다) **10종 중 6종
(`GERM_TYPE_COUNT`)을 Fisher–Yates 셔플로 무작위로 골라** 사용한다. 매 촬영마다 다른
조합이 나오도록 하기 위함이며, 하드코딩된 고정 6종이 아니다. [[Germ_Display_Mode|캐릭터
모드]]에서는 이 선택 자체가 무시된다(항상 벡터 도형).
