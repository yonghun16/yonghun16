# 04. 에셋 관리 및 개발 로드맵 (Assets & Execution Roadmap)

## 🔗 연관 문서
- [[00_Overview]]
- [[01_Architecture_Flow]]
- [[03_Germ_and_Clean_Rendering]]

---

## 📁 1. 에셋 폴더 구조
현재 `assets/` 아래에 실제로 존재하는 파일과, 아직 없는 `effects/`·`sounds/`의 상태.
→ [[Asset_Folder_Structure]]

## 🎨 2. 세균/반짝이 컴포넌트 이원화 구조 (Dummy Fallback)
PNG 에셋이 없거나(반짝이) 사용자가 캐릭터 모드를 선택했을 때, 같은 컴포넌트가
벡터 도형으로 자연스럽게 폴백하는 구조. [[Germ_Display_Mode|캐릭터 모드]]의 기반이기도 하다.
→ [[Dummy_Component_Fallback]]

---

## 🚀 3. 개발 로드맵

### Phase 1~5 (초기 구현, 완료)
- **Phase 1**: 프로젝트 초기 세팅, 의존성 설치, 에셋 이원화 컴포넌트(`DummyGerm.tsx`) 구성
- **Phase 2**: `CameraScreen` 구현 및 `washMode` 모드 선택 UI 구성
- **Phase 3**: 온디바이스 손 좌표 추출 파이프라인 연동, `analyzeHandImage.ts` 및 예외 처리
- **Phase 4**: Skia Canvas 세균/반짝이 오버레이, 우하단 히든 버튼 구현
- **Phase 5**: Pinch Zoom 제스처 연동, 효과음/진동 연출 결합

### Phase 6+ (실기기 테스트 기반 반복 개선, 완료)
Phase 5 완료("전체 흐름 마무리") 이후에도 보건교사의 실제 사용 시나리오를 가정한
실기기(Android 폴더블 기기 + 릴리즈 APK) 테스트를 계속하며 아래 항목들을 추가/수정했다.
자세한 근거와 구현 디테일은 링크된 노드를 참고할 것.

- **안정성**: 실기기 크래시 원인이던 `expo-av` 제거, [[Release_Build_Asset_Loading|릴리즈 빌드 전용 TFLite 모델 로딩 크래시]] 수정, 릴리즈 APK 빌드/배포 확립
- **인식 정확도**: 단순 크롭 방식 → [[Two_Stage_Hand_Pipeline|palm detector 기반 2단계 파이프라인]]으로 교체해 "손 미인식" 문제 해결
- **촬영 품질**: [[Camera_Capture_and_Crop|미리보기-사진 화각 불일치 보정]]
- **세균**: [[Germ_Animations|리빌 효과와 숨쉬기 애니메이션]], [[Microscope_Haze_Layer|현미경 뿌연 조명]], 실제 세균 사진 10종 + 무작위 6종 선택, 개수 확대(15~30 → 55~90, [[Germ_Generation_Algorithm]])
- **Pinch Zoom 아키텍처 전환**: CSS transform → [[Rendering_Architecture|Skia 내부 transform]] (고배율 확대 시 흐려지는 문제 해결), 최대 배율 4x → 10x(현미경 모드, [[Pinch_Zoom_Gesture]])
- **AFTER 모드 연출**: 반짝이 트윙클 애니메이션, 반짝이 모양(원 → 4방향 별)과 배치 자연화, 손 마스크 라이트 스윕 → 화면 전체 플래시로 교체 ([[After_Mode_Effects]])
- **화면 구조 확장**: Home 화면과 Settings 화면 신설, [[Germ_Display_Mode|균 표시 방식(현미경/캐릭터 모드)]] 설정 추가 및 [[Settings_and_Persistence|AsyncStorage 영구 저장]]
- **UI 다듬기**: `@expo/vector-icons` 도입, 텍스트 글리프/이모지 아이콘을 벡터 아이콘으로 교체, washMode 전환을 상단 탭 → 우하단 반투명 원형 토글로 변경, 종료 버튼 → 뒤로가기 버튼으로 교체, [[Hand_Not_Detected_Modal|손 미인식 알림을 커스텀 카드형 모달로 교체]]
