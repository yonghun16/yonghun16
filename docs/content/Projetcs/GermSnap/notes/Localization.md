# 다국어 지원 (Localization)

## 🔗 상위 문서
[[00_Overview]]

## 관련 노드
[[Settings_and_Persistence]]

---

기기 언어 설정에 따라 앱 안의 모든 화면 문구가 자동으로 번역되어 보이도록
`i18next` + `react-i18next` + `expo-localization`을 사용한다.

## 지원 언어
- 영어(`en`, 기본값/폴백) · 한국어(`ko`) · 일본어(`ja`) · 중국어 간체(`zh`)
- 지원하지 않는 기기 언어(예: 프랑스어, 독일어)는 영어로 표시된다.

## 구현 (앱 내부 UI)
- `src/i18n/locales/{ko,en,ja,zh}.json` — 화면별로 중첩된 번역 리소스
  (`common`, `home`, `settings`, `help`, `camera`, `result` 네임스페이스)
- `src/i18n/index.ts` — `expo-localization`의 `Localization.getLocales()[0]`으로
  기기의 1순위 언어 코드를 읽어, 지원 언어 목록에 있으면 그 언어로,
  없으면 영어(`DEFAULT_LANGUAGE`)로 `i18next`를 초기화한다. `App.tsx`가
  최상단에서 `import './src/i18n'`로 부팅 시점에 한 번 초기화한다.
- 각 화면(`HomeScreen`, `SettingsScreen`, `HelpScreen`, `CameraScreen`,
  `ResultScreen`)은 `useTranslation()`의 `t('namespace.key')`로 문자열을
  가져온다. 하드코딩된 한국어 문자열은 남기지 않는 것이 원칙이다.
- 앱 이름이 화면에 노출되는 곳(`home.title`, `help.subtitle`)은 언어별로
  브랜드명을 그대로 번역해 둔다 — 영어 "Germ Snap"(띄어쓰기 포함), 한국어
  "세균 카메라", 일본어 "細菌カメラ", 중국어 "细菌相机". 아래 "앱 런처 표시
  이름"과 다른 메커니즘(하나는 i18next, 하나는 네이티브 문자열 리소스)이지만
  같은 이름이 나오도록 맞춰뒀다.

## 구현 (앱 런처 표시 이름)
JS 번들 밖의 네이티브 영역이라 위 `i18next` 체계와는 별도로,
`@expo/config-plugins`의 `withLocales` 플러그인을 이용한다.
- `app.json`의 최상위 `"name"`이 기본 표시 이름(모든 언어의 폴백) — `"Germ Snap"`
  (영어 표기 시 반드시 띄어쓰기 포함, `"GermSnap"` 아님).
- `app.json`의 `"locales"` 필드가 언어 코드 → 오버라이드 JSON 파일 경로를 매핑한다.
  현재는 `"ko": "./locales/ko.json"` 하나만 있고, 그 파일 안의 `app_name`
  (Android) / `CFBundleDisplayName`(iOS) 키가 한국어 기기에서 "세균 카메라"로
  표시되도록 오버라이드한다.
- `app.json`을 수정한 뒤에는 `expo prebuild --platform android --clean`으로
  네이티브 프로젝트를 다시 생성해야 반영된다 (Android는
  `android/app/src/main/res/values-b+ko/strings.xml`에 오버라이드가 생성됨).
  prebuild `--clean`은 `android/gradle.properties`의 커스텀 JVM 힙 크기
  설정도 초기화하므로 재적용이 필요하다.

## ⚠️ 알려진 범위 제한
- OS 카메라 권한 요청 팝업의 안내 문구(`app.json`의 `cameraPermission`)는
  여전히 한국어로 고정되어 있다. 필요해지면 위와 같은 `"locales"` 오버라이드
  방식으로 언어별 문구를 추가할 수 있다. (앱 안의 카메라 권한 유도 화면 문구는
  이미 `i18next`로 번역됨.)
- 앱 런처 표시 이름은 한국어(`ko`)만 `"locales"`로 오버라이드되어 있다.
  일본어/중국어 기기에서는 런처 이름이 여전히 "Germ Snap"으로 나오는데,
  앱을 실행하면 안의 UI는 "細菌カメラ"/"细菌相机"로 보인다 — 필요하면
  `locales/ja.json`, `locales/zh.json`을 추가해 런처 이름도 맞출 수 있다.

## 새 문구를 추가할 때
새 화면/문구를 추가할 때는 `t('...')` 없이 문자열을 하드코딩하지 말고, 반드시
4개 로케일 파일 모두에 키를 추가할 것 — 한 언어만 추가하면 다른 언어에서는
`i18next`의 `fallbackLng: 'en'`에 의해 영어로 대체 표시된다(깨지진 않지만
번역 누락 상태로 남는다).
