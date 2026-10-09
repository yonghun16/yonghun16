# 손 미인식 예외 처리 및 커스텀 모달

## 🔗 상위 문서
[[02_Camera_and_AI]]

## 관련 노드
[[Two_Stage_Hand_Pipeline]]

---

## 트리거 조건
[[Two_Stage_Hand_Pipeline|2단계 파이프라인]]이 `null`을 반환하거나(손이 인식되지 않음)
내부에서 예외가 발생한 경우. `CameraScreen`은 두 경우를 사용자에게는 동일하게
보여주지만, 원인 파악을 위해 `console.error`로 콘솔에는 실제 에러를 남긴다 (이 로그가
[[Release_Build_Asset_Loading]] 문제를 실기기에서 진단하는 데 결정적이었다).

## UI: OS 기본 Alert가 아니라 커스텀 카드형 모달
초기 구현은 `Alert.alert(title, message)`였지만 "기본 Alert는 촌스럽다"는 피드백에
따라 앱 톤에 맞는 모달로 교체했다 (`isHandNotDetectedVisible` 상태로 제어).

구성 요소:
- 둥근 원형 배지 안에 `Ionicons name="hand-left-outline"` 아이콘 (연주황 배경)
- 제목: "손이 보이지 않아요!"
- 안내 문구: "손바닥이 화면 중앙에 크게 오도록 다시 찍어주세요!"
- "다시 찍기" 버튼 (파란 필 버튼) → 모달 닫힘, `CameraScreen`에 그대로 머무름 (별도
  화면 전환 없음)

진동(`Haptics.NotificationFeedbackType.Error`)과 함께 노출된다.

> 같은 "OS 기본 UI → 커스텀 카드형 모달" 교체 패턴이 카메라 화면의 종료 확인에도
> 쓰인 적이 있었지만, 이후 종료 버튼 자체가 뒤로가기 버튼으로 대체되며 삭제됐다
> (요즘 앱은 별도 종료 버튼을 두지 않는 것이 트렌드라는 피드백에 따름).
