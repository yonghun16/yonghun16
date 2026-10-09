# 03. 세균/이펙트 렌더링 & 제스처 (Germ Rendering & Interaction)

## 🔗 연관 문서
- [[01_Architecture_Flow]]
- [[02_Camera_and_AI]]
- [[04_Asset_and_Development_Guide]]

이 문서는 `ResultScreen`이 그리는 요소들을 개괄한다. 각 항목의 상세 구현/알고리즘/
이력은 아래 노드를 참고할 것.

---

## 🖼️ 렌더링 아키텍처
사진·세균·반짝이를 모두 Skia `Canvas` 안에서 그리고, Pinch Zoom도 CSS transform이
아닌 Skia 내부 `Group transform`으로 처리한다 (고배율 확대 시 흐려지는 문제 방지).
→ [[Rendering_Architecture]]

## 🔬🦠 균 표시 방식 (현미경 모드 / 캐릭터 모드)
사실적인 세균 사진을 무서워하는 아이들을 위해, 설정에서 고를 수 있는 두 가지 표시
방식. 최대 확대 배율과 haze 레이어 표시 여부가 함께 갈린다.
→ [[Germ_Display_Mode]]

## 🦠 세균 그래픽 오버레이 (`washMode === 'BEFORE'`)
- 생성 개수/위치/회전/스케일 알고리즘 → [[Germ_Generation_Algorithm]]
- 확대해야 보이는 리빌 효과 + 숨쉬기 애니메이션 → [[Germ_Animations]]
- 현미경 뿌연 조명(haze) 레이어 → [[Microscope_Haze_Layer]]

## ✨ 깨끗해진 손 이펙트 (`washMode === 'AFTER'`)
반짝이 배치/모양, 트윙클 애니메이션, 화면 전체 플래시, 칭찬 메시지.
→ [[After_Mode_Effects]]

## 🔍 확대/축소 제스처 (Pinch Zoom + Pan)
`react-native-gesture-handler` 기반 제스처 구현과 모드별 최대 배율.
→ [[Pinch_Zoom_Gesture]]

## 🤫 우하단 히든 버튼 (Clean Mode Toggle)
- **위치**: `ResultScreen` 우하단 구석 (가로 30px x 세로 30px)
- **스타일**: 거의 안 보이는 투명도 (`opacity: 0.03`)
- **기능**: 탭 시 `isCleanMode` 상태를 토글 ➔ `true`가 되면 BEFORE 모드로 찍힌
  사진이라도 세균 레이어의 `opacity`를 `0`으로 전환해 깨끗해진 손처럼 보이게 한다
  (보건교사가 시연용으로 강제 전환할 때 사용).
