# 01. 시스템 구조 및 화면 흐름 (Architecture & Flow)

## 🔗 연관 문서
- [[00_Overview]]
- [[02_Camera_and_AI]]
- [[03_Germ_and_Clean_Rendering]]

---

## 🔄 핵심 상태 (State)

앱 전역 상태는 `App.tsx`가 들고 있으며, 성격이 다른 두 종류로 나뉜다.

### 1. 촬영 세션 상태 (`AppState`) — 사진 한 장을 찍고 다시 찍을 때마다 초기화됨
```typescript
interface AppState {
  washMode: 'BEFORE' | 'AFTER';  // 현재 촬영 모드
  photoUri: string | null;        // 촬영된 정지 이미지 경로
  handLandmarks: Point3D[];       // MediaPipe 파이프라인이 반환한 21개 좌표
  isCleanMode: boolean;           // 우하단 히든 버튼 클릭 시 토글되는 상태
}
```

### 2. 앱 설정 상태 — 화면 전환과 무관하게 유지/영구 저장됨
- `hasStarted: boolean` — Home 화면인지, Camera/Result 화면으로 넘어갔는지
- `homeView: 'main' | 'settings' | 'help'` — Home 화면 위에 Settings/Help 중 무엇을
  보여줄지 (셋 중 하나만 배타적으로 열림)
- `germDisplayMode: 'MICROSCOPE' | 'CHARACTER'` — 균 표시 방식. 앱 시작 시
  `AsyncStorage`에서 불러오고, 설정 화면에서 바꿀 때마다 즉시 저장한다
  → [[Settings_and_Persistence]], [[Germ_Display_Mode]]

---

## 📱 화면 전환 흐름 (Screen Flow)

```text
[HomeScreen] (앱 시작 시 항상 여기부터)
   ├─ 은은하게 비치는 배경 일러스트(assets/pic.png) + 반투명 흰색 오버레이
   ├─ 아이콘 / 타이틀 / 소개 문구
   ├─ 안내 문구(작게) — "실제 세균이 아닌, 교육용으로 합성한 이미지예요"
   │    도움말까지 찾아 들어가지 않아도 시작 전에 누구나 보도록 인트로에 바로 배치
   ├─ [시작하기] 버튼 ─────────────────────► hasStarted = true ──► [CameraScreen]
   └─ 보조 버튼 2개 (시작하기 아래, 나란히)
        ├─ [⚙ 설정] ──────────────────────► homeView = 'settings' ──► [SettingsScreen]
        └─ [❓ 도움말] ────────────────────► homeView = 'help' ──► [HelpScreen]

[SettingsScreen]
   ├─ [‹ 뒤로가기] ────────────────────────────────────────────► [HomeScreen]
   └─ "균 표시 방식" 카드 2개 (현미경 모드 / 캐릭터 모드)
        └─ 탭하면 germDisplayMode 즉시 변경 + AsyncStorage에 영구 저장

[HelpScreen]
   ├─ [‹ 뒤로가기] ────────────────────────────────────────────► [HomeScreen]
   └─ 사용법 안내 카드 목록 (스크롤) — 촬영, washMode 전환, 확대, 다시 찍기, 설정
        보건교사용 히든 버튼처럼 의도적으로 숨긴 기능은 여기서 설명하지 않는다.
        (교육용 합성 이미지 고지는 HelpScreen이 아니라 HomeScreen에 있음 — 위 참고)

[CameraScreen]
   ├─ [‹ 뒤로가기] (좌상단) ───────────────────────────────────► hasStarted = false ──► [HomeScreen]
   ├─ washMode 토글 (우하단, 반투명 원형 아이콘 버튼 — 분홍 🧼 BEFORE / 하양 ✨ AFTER)
   │    보건교사가 손이나 몸으로 살짝 가려 탭할 수 있도록 설명 텍스트 없이 아이콘만 둠
   ├─ 손 모양 점선 가이드라인 오버레이
   └─ [촬영] 버튼 클릭
         │
         ▼ (사진 crop → Photo URI 확보)
   [스캔 로딩 오버레이] (약 0.8초 + 실제 추론 시간, "스캔 중..." + 진동)
         │
         ▼ (2단계 온디바이스 파이프라인으로 손 좌표 분석, [[Two_Stage_Hand_Pipeline]])
   ┌─────────────┴─────────────┐
   │ (손 인식 성공)             │ (손 미인식 또는 내부 오류)
   ▼                           ▼
[ResultScreen]              [손 미인식 모달] (카드형 커스텀 모달, 기본 Alert 아님)
                                └─► [다시 찍기] 버튼 ──► CameraScreen에 머무름

[ResultScreen]
   ├─ Skia Canvas: 사진 + (BEFORE) 세균 오버레이 또는 (AFTER) 반짝이 + 화면 전체 플래시
   ├─ Pinch Zoom + Pan (germDisplayMode에 따라 최대 배율이 다름: 현미경 10x / 캐릭터 4x)
   ├─ [↺ 다시 찍기] 버튼 (좌상단) ─────────────────────────────► photoUri = null ──► [CameraScreen]
   └─ [히든 버튼 🤫] (우하단, opacity 0.03) ➔ isCleanMode 강제 토글
```

## 📌 화면 컴포넌트 매핑

| 화면 | 파일 |
| :--- | :--- |
| Home | `src/screens/HomeScreen.tsx` |
| Settings | `src/screens/SettingsScreen.tsx` |
| Help | `src/screens/HelpScreen.tsx` |
| Camera | `src/screens/CameraScreen.tsx` |
| Result | `src/screens/ResultScreen.tsx` |
| 최상위 상태/라우팅 | `App.tsx` (별도 네비게이션 라이브러리 없이 조건부 렌더링으로 화면을 전환) |
