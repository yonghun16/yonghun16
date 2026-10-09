# 02. 카메라 및 AI 손 인식 로직 (Camera & On-Device AI)

## 🔗 연관 문서
- [[01_Architecture_Flow]]
- [[03_Germ_and_Clean_Rendering]]

이 문서는 카메라 촬영부터 손 좌표 추출까지의 흐름을 개괄한다. 각 항목의 상세
구현/함정/이력은 아래 노드를 참고할 것.

---

## 📸 1. 카메라 촬영 및 화각 보정
`CameraScreen.tsx`가 손 모양 가이드라인을 오버레이한 미리보기를 보여주고, 촬영 후
미리보기와 동일한 화각으로 사진을 크롭한다.
→ [[Camera_Capture_and_Crop]]

## 🤖 2. On-Device 손 좌표 추출 파이프라인
공식 `react-native-mediapipe`가 손 랜드마크를 지원하지 않아, palm detector + hand
landmark 두 TFLite 모델을 직접 구동하는 2단계 파이프라인을 자체 구현했다.
→ [[Two_Stage_Hand_Pipeline]]

### ⚠️ 릴리즈 빌드 전용 함정
디버그 빌드에서는 되는데 릴리즈 빌드에서만 TFLite 모델 로딩이 크래시하는 문제와 그
해결 방법.
→ [[Release_Build_Asset_Loading]]

## ⚠️ 3. 예외 처리
손이 인식되지 않거나 파이프라인 내부 오류가 발생했을 때의 UI/UX (OS 기본 Alert가
아니라 커스텀 카드형 모달).
→ [[Hand_Not_Detected_Modal]]
