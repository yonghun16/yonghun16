# 00. 프로젝트 개요 (Overview)

## 📌 프로젝트 정보
- **앱 이름**: Germ Snap (영어 등 기본값) / "세균 카메라"(한국어 기기) / "細菌カメラ"(일본어) / "细菌相机"(중국어) — 런처 이름은 한국어만 오버라이드되어 있고, 앱 내부 UI 텍스트는 4개 언어 모두 이 이름을 쓴다 → [[Localization]]
- **목적**: 초등학교 어린이 보건교육을 위한 100% 오프라인 On-Device AI 손 세균 시각화 모바일 앱
- **타겟 사용자**: 초등학생 및 보건교사 (보건교사가 기기를 조작하고, 아이들이 화면을 봄)
- **패키지명**: `com.anonymous.handgermscanner` (배포 전 변경 예정 — 아직 미확정)
- **저장소**: `github.com/yonghun16/GermSnap`

> ⚠️ 이 문서는 최초 기획 초안이 아니라, 실기기 테스트와 사용자 피드백을 거쳐 여러 차례
> 개정된 **현재 구현 상태의 명세서**입니다. Phase 1~5(초기 구현) 완료 이후에도 실기기
> 버그 수정과 기능 추가가 계속 있었으므로, 코드를 수정하기 전 반드시 이 문서들을 먼저
> 확인하세요.

## 🔗 연관 문서 (Obsidian Links)
- 시스템 구조 및 화면 흐름: [[01_Architecture_Flow]]
- 카메라 및 AI 손 인식 로직: [[02_Camera_and_AI]]
- 세균/이펙트 렌더링 & 제스처: [[03_Germ_and_Clean_Rendering]]
- 에셋 관리 및 개발 로드맵: [[04_Asset_and_Development_Guide]]

### 📂 문서 구조 (허브 + 노드)
`00_Overview` ~ `04_Asset_and_Development_Guide` 5개 문서는 화면/영역별 **허브(MOC)**
문서로, 각 주제의 개요만 담고 상세 내용은 `docs/content/notes/` 아래의 **개별 노드**로
링크한다 (Obsidian 그래프 뷰에서 허브→노드, 노드→노드 관계가 드러나도록 하기 위함).
새 기능을 문서화할 때는 관련 있는 허브에서 링크로 참조되는 새 노드를 `notes/`에
추가하는 방식을 기본으로 한다 — 기존 허브 문서 본문을 계속 늘리지 않는다.

---

## 🛠️ 기술 스택 (Tech Stack)

| 구분 | 기술 / 라이브러리 | 용도 |
| :--- | :--- | :--- |
| **Framework** | React Native 0.86 (Expo Bare Workflow, New Architecture) | Cross-platform 모바일 앱. `android/`, `ios/`는 `.gitignore` 처리되어 `expo prebuild`로 계속 재생성하는 구조 |
| **Runtime** | Expo SDK 57 | `expo-camera`, `expo-asset`, `expo-image-manipulator`, `expo-haptics`, `expo-status-bar` 등 |
| **Language** | TypeScript | 타입 안전성 확보 |
| **On-Device AI** | `react-native-fast-tflite` (Nitro Modules) + 자체 구현 2단계 파이프라인 | palm 검출 → 손 회전 정렬 크롭 → 21개 손 좌표 landmark 추출 (완전 오프라인) |
| **2D Graphics** | `@shopify/react-native-skia` | 사진/세균/반짝이/현미경 이펙트를 하나의 Canvas에 렌더링 |
| **Gesture & Animation** | `react-native-gesture-handler`, `react-native-reanimated` + `react-native-worklets` | Pinch Zoom + Pan, 세균 숨쉬기, 반짝이 트윙클, AFTER 모드 플래시 애니메이션 |
| **Camera** | `expo-camera` | 카메라 미리보기 및 정지 이미지 캡처 |
| **이미지 후처리** | `expo-image-manipulator` | 촬영된 사진을 미리보기와 동일한 비율로 네이티브 크롭 |
| **아이콘** | `@expo/vector-icons` (Ionicons, MaterialCommunityIcons) | 뒤로가기/다시 찍기/설정 등 UI 아이콘 |
| **저장소** | `@react-native-async-storage/async-storage` | 균 표시 방식(현미경/캐릭터) 설정을 기기에 영구 저장 |
| **다국어** | `i18next`, `react-i18next`, `expo-localization` | 기기 언어(한국어/영어/일본어/중국어)에 맞춰 앱 UI 문구 자동 번역 → [[Localization]] |
| **인앱 영상** | `react-native-youtube-iframe` (+ `react-native-webview`) | Home 화면에서 언어권별 손 씻기 영상을 유튜브 앱 대신 인앱 전체화면 임베드로 재생 → [[Handwash_Video]] |
| **Feedback** | `expo-haptics` | 스캔/성공/실패 진동. 효과음(`sound.ts`)은 자리만 마련된 상태이며 현재는 no-op (아래 "알려진 갭" 참고) |

---

## 💡 개발 핵심 원칙
1. **100% 오프라인 동작**: 서버 통신 코드 금지 (학교 현장의 Wi-Fi 불안정 고려).
   유일한 예외는 Home 화면의 "올바른 손 씻기 영상" 버튼 — 언어권별 YouTube
   영상을 인앱 전체화면으로 자동 재생하는 기능으로, 영상 재생 자체가 인터넷을
   전제로 하는 선택적 부가 기능이라 예외로 둔다 (핵심 스캔 기능은 여전히 완전
   오프라인) → [[Handwash_Video]].
2. **단일 정지 이미지 분석**: 실시간 동영상 분석 대신, 촬영된 정지 이미지 한 장을 분석하는 방식 유지
3. **명세서 우선**: `docs/content/`의 명세를 먼저 확인하고 코드를 수정/작성한다. 명세와 실제 코드가 다르다면, 실기기 검증을 거친 코드 쪽이 최신 진실이며 이 문서를 갱신해야 한다는 뜻이다.

## ⚠️ 알려진 갭 (아직 안 끝난 부분)
- **효과음**: `assets/sounds/`에 실제 mp3 파일이 없어 `src/utils/sound.ts`의 `playScanSound`/`playCleanSound`/`playErrorSound`는 현재 no-op이다. `expo-av`는 실기기에서 네이티브 크래시(UnsatisfiedLinkError)를 일으켜 제거했고, mp3 에셋이 준비되면 `expo-audio`로 교체할 계획이다 ([[04_Asset_and_Development_Guide]] 참고).
