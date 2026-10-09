# 카메라 촬영 및 미리보기-사진 화각 보정

## 🔗 상위 문서
[[02_Camera_and_AI]]

## 관련 노드
[[Two_Stage_Hand_Pipeline]]

---

## 촬영 흐름 (`CameraScreen.tsx`)
1. `expo-camera`의 `CameraView`로 미리보기를 표시하고, 화면 중앙에 손 모양 점선
   가이드라인(`GUIDE_RECT`)을 오버레이한다. 팜 검출기가 손 위치/크기/회전을 자동으로
   찾아주므로 이 가이드는 실제 크롭 영역이 아니라 "손을 화면 안에 적당히 크게
   담아달라"는 시각적 안내 용도다.
2. [촬영] 버튼 탭 시 `takePictureAsync({ quality: 0.8 })`로 정지 이미지를 얻는다.
3. 약 800ms(`SCAN_OVERLAY_DURATION_MS`)간 "스캔 중..." 오버레이 + 진동을 보여준 뒤
   [[Two_Stage_Hand_Pipeline|손 좌표 분석]]을 시작한다.

## 문제: 미리보기와 실제 사진의 화각이 다르다
`expo-camera` 미리보기는 화면을 꽉 채우기 위해 `cover` 방식으로 잘라서 보여주지만,
`takePictureAsync()`로 찍히는 실제 사진은 카메라 센서 원본 비율 그대로라 미리보기보다
화각이 더 넓게 나온다 ("더 멀리서 찍은 것처럼" 보이는 문제, 실기기에서 확인됨).

## 해결: 네이티브 크롭
`src/utils/cropToAspectRatio.ts`의 `cropPhotoToAspectRatio()`가 미리보기 컨테이너의
가로/세로 비율(`previewSize`, `onLayout`으로 측정)에 맞춰 촬영된 사진을 중앙 크롭한다.

```typescript
export const cropPhotoToAspectRatio = async (
  uri: string,
  photoWidth: number,
  photoHeight: number,
  targetAspectRatio: number
): Promise<string> => { /* expo-image-manipulator의 manipulateAsync(crop)로 구현 */ };
```

### 왜 `expo-image-manipulator`인가 (Skia 크롭을 폐기한 이유)
최초 구현은 Skia로 원본 해상도 전체를 디코드 → 크롭 → `encodeToBase64()`하는
방식이었는데, 실기기에서 수 초간 "스캔 중..."에 멈춘 것처럼 보이는 **행(hang) 문제**가
있었다. JS 스레드는 살아있었지만(ActivityIndicator는 계속 애니메이션 중) Promise
체인이 끝나지 않았고, 원인은 고해상도 사진의 무거운 base64 왕복이었다. 네이티브
`expo-image-manipulator`의 크롭으로 교체해 해결했다.
