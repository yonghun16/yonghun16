# 올바른 손 씻기 영상 (인앱 재생)

## 🔗 상위 문서
[[00_Overview]]

## 관련 노드
[[Localization]]

---

Home 화면의 "올바른 손 씻기 영상" 버튼(`src/screens/HomeScreen.tsx`)을 누르면
언어권에 맞는 손 씻기 안내 유튜브 영상을 앱 안에서 전체화면으로 자동 재생한다.

## 왜 유튜브 앱이 아니라 인앱 재생인가
처음엔 `Linking.openURL()`로 유튜브 앱을 여는 방식이었다. 하지만 이 앱은
교실에서 초등학생이 보는 화면이라, 유튜브 앱으로 나가면 추천 영상·댓글·다음
영상 자동재생 등으로 아이들이 관련 없는 콘텐츠로 새어나갈 위험이 있다.
`react-native-youtube-iframe`으로 유튜브 iframe 임베드 플레이어를 앱 안의
전체화면 모달에 띄우는 방식으로 바꿔서, 딱 그 영상 하나만 보여주고 끝낼 수
있게 했다.

## 임베드 가능 여부 사전 확인
유튜브 영상은 업로더가 "퍼가기(임베드) 허용"을 꺼둔 경우 외부 플레이어에서
재생이 안 될 수 있다. 4개 영상 모두 아래처럼 YouTube oEmbed 엔드포인트로
사전에 확인했다 (200 OK = 임베드 가능, 401 = 임베드 차단):

```
https://www.youtube.com/oembed?url=https://www.youtube.com/watch?v=<영상ID>&format=json
```

4개 모두 핑크퐁(Baby Shark)의 손 씻기송 시리즈이며 임베드 가능하다.

| 언어 | 영상 ID | 제목 |
| :--- | :--- | :--- |
| 한국어 | `JvfyAtvZRvk` | 아기상어와 6단계 손 씻기 |
| 영어 | `L89nN03pBzI` | Wash Your Hands with Baby Shark |
| 일본어 | `0BjE3_Xl8t0` | ちびザメとてをあらおう |
| 중국어 | `AR0kNEb0aq0` | 和鯊魚寶寶一起洗洗手 |

## 구현
- `HANDWASH_VIDEO_IDS`(`HomeScreen.tsx`) — [[Localization|현재 앱 언어]]
  (`i18n.language`)에 따라 영상 ID를 고른다. 지원하지 않는 언어는 한국어로 폴백.
- `isVideoVisible` 상태로 전체화면 `Modal`을 띄우고, 그 안에
  `react-native-youtube-iframe`의 `<YoutubePlayer videoId={...} play />`를
  렌더링한다. 이 라이브러리가 내부적으로 자체 검증된 iframe 호스트 HTML/JS
  브리지를 통해 `WebView`를 구성해주므로, 유튜브의 까다로운 임베드 검증
  (정상적인 `<iframe>` 컨텍스트, 부모 문서의 출처 등)을 직접 신경 쓸 필요가
  없다.
  - `forceAndroidAutoplay`로 일부 안드로이드 기기에서 자동재생이 막히는
    문제를 우회한다.
  - `initialPlayerParams={{ rel: false }}`로 재생 종료 후 추천 영상이 다른
    채널로 새지 않게 제한한다.
  - `height`/`width`는 `useWindowDimensions()`로 구한 화면 너비 기준
    16:9 비율로 계산해서 넘긴다.
- 플레이어는 `isVideoVisible`이 `true`일 때만 마운트한다 — `Modal`만
  숨기고 계속 살려두면 닫은 뒤에도 백그라운드에서 소리가 날 수 있어서,
  닫을 때 아예 언마운트되도록 조건부 렌더링했다.
- 우상단 반투명 원형 X 버튼으로 닫는다.

### ⚠️ 함정: "오류 153 동영상 플레이어 구성 오류" (직접 만든 WebView 방식에서 발생, 라이브러리 교체로 해결)
처음엔 `WebView`에 직접 유튜브 embed URL/HTML을 넣는 방식으로 구현했는데,
두 가지 원인이 겹쳐 계속 오류 153이 났다:
1. **iframe 없이 embed URL을 직접 로드**하면 유튜브가 "정상적인 `<iframe>`
   안에 있지 않다"고 판단해 재생을 거부한다.
2. 로컬 HTML 문자열을 `baseUrl` 없이 로드하면 문서의 출처(origin)가
   비어있는 상태(opaque origin)가 되어, 부모 페이지의 출처를 확인하지
   못하는 유튜브 임베드 검증에 또 걸린다.

`<iframe>`으로 감싼 HTML을 만들고 `baseUrl: 'https://www.youtube.com'`을
지정하는 두 가지 수정을 각각 적용해봤지만 실기기에서 오류가 계속
재현됐다 — 유튜브의 임베드 검증(User-Agent, referrer 등)이 자체 구현
HTML/iframe 방식으로는 계속 놓치는 지점이 있었던 것으로 보인다. 이 문제를
전문적으로 다루는 검증된 라이브러리 `react-native-youtube-iframe`으로
교체하면서 완전히 해결됐다 (실기기에서 정상 재생 확인).

## ⚠️ 오프라인 원칙의 예외
[[00_Overview]]의 "100% 오프라인 동작" 원칙에서 유일하게 예외인 기능이다 —
영상 재생 자체가 인터넷 연결을 전제로 하는 선택적 부가 기능이며, 핵심
스캔 기능(카메라 촬영, 손 인식, 세균 렌더링)은 여전히 완전 오프라인으로
동작한다.
