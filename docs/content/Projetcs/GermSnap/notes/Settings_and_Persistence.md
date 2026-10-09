# 설정 화면 및 영구 저장

## 🔗 상위 문서
[[01_Architecture_Flow]]

## 관련 노드
[[Germ_Display_Mode]] · [[Pinch_Zoom_Gesture]] · [[Microscope_Haze_Layer]]

---

## 화면 구조
`HomeScreen`의 "시작하기" 버튼 아래에 보조(아웃라인 스타일) "⚙ 설정" 버튼이 있다.
탭하면 `App.tsx`의 `isSettingsOpen` 상태가 `true`가 되어 `SettingsScreen`이
Home 화면 자리에 대신 렌더링된다 (별도 네비게이션 스택 없이 조건부 렌더링).

`SettingsScreen`에는 "균 표시 방식" 섹션이 있고, [[Germ_Display_Mode|현미경 모드 /
캐릭터 모드]] 카드 두 개를 보여준다. 카드를 탭하면 즉시 적용되고(별도 저장 버튼
없음), 선택된 카드에는 파란 테두리 + 체크 아이콘이 표시된다.

## 영구 저장 (AsyncStorage)
사용자가 "균 표시 방식 설정을 앱을 껐다 켜도 계속 기억해줬으면 좋겠다"고 요청해서,
`@react-native-async-storage/async-storage`를 새로 설치해 기기에 저장한다
(`src/utils/germDisplayModeStorage.ts`).

```typescript
const STORAGE_KEY = 'handGermScanner.germDisplayMode';

export const loadGermDisplayMode = async (): Promise<GermDisplayMode> => { /* AsyncStorage.getItem */ };
export const saveGermDisplayMode = async (mode: GermDisplayMode): Promise<void> => { /* AsyncStorage.setItem */ };
```

`App.tsx`는 마운트 시 `loadGermDisplayMode()`로 저장된 값을 불러와
`germDisplayMode` 상태의 초기값으로 쓰고(기본값은 `'CHARACTER'`), 설정 화면에서
값이 바뀔 때마다 즉시 `saveGermDisplayMode()`로 저장한다. 이 상태는
`ResultScreen`에 prop으로 전달되어 [[Pinch_Zoom_Gesture|최대 확대 배율]]과
[[Microscope_Haze_Layer|haze 레이어 표시 여부]]를 함께 결정한다.

> ℹ️ `germDisplayMode`는 촬영 세션 상태(`AppState`: `washMode`/`photoUri`/
> `handLandmarks`/`isCleanMode`)와 달리, 사진을 다시 찍어도 초기화되지 않는
> 앱 전역 설정이라 `AppState`에 포함하지 않고 별도 상태로 관리한다.
