# 릴리즈 빌드 전용 함정: TFLite 모델 에셋 로딩

## 🔗 상위 문서
[[02_Camera_and_AI]]

## 관련 노드
[[Two_Stage_Hand_Pipeline]]

---

## 증상
`loadTensorflowModel(require('...tflite'))`은 디버그 빌드(Metro가 HTTP로 서빙)에서는
정상 동작하지만, 릴리즈 빌드에서는 아래 런타임 크래시를 일으킨다 (실기기에서 확인됨).

```
MalformedURLException: no protocol: assets_models_palm_detection_lite
```

Expo의 릴리즈 에셋 번들링이 만들어내는 참조 문자열을 `react-native-fast-tflite`
네이티브 코드가 URL로 파싱하지 못해서 생기는 문제다.

## 진단 방법
`CameraScreen.tsx`의 catch 블록에 `console.error`를 추가하고 `adb logcat`으로 확인해서
찾았다. 그전까지는 "손 미인식"과 실제 오류가 사용자에게 동일한 메시지로 뭉뚱그려져
있어 원인 파악이 어려웠다.

## 해결
`src/utils/tfliteModel.ts`는 `expo-asset`의 `Asset.fromModule(module).downloadAsync()`로
실제 `file://` URI를 구한 뒤 `loadTensorflowModel({ url })`로 로드한다.

```typescript
const resolveModelUri = async (assetModule: number): Promise<string> => {
  const asset = await Asset.fromModule(assetModule).downloadAsync();
  if (!asset.localUri) throw new Error('모델 애셋의 로컬 경로를 확인할 수 없습니다.');
  return asset.localUri;
};
```

> ⚠️ **모델 로딩 방식을 건드릴 때는 반드시 릴리즈 APK로도 검증할 것.** 디버그 빌드에서만
> 테스트하면 이 문제를 재현할 수 없다.
