# 2단계 온디바이스 손 좌표 추출 파이프라인

## 🔗 상위 문서
[[02_Camera_and_AI]]

## 관련 노드
[[Release_Build_Asset_Loading]] · [[Camera_Capture_and_Crop]] · [[Hand_Not_Detected_Modal]]

---

공식 `react-native-mediapipe` 패키지는 손 랜드마크를 지원하지 않는다 (오브젝트/포즈/얼굴만
지원하고, `expo-camera`가 아닌 `vision-camera`가 필요). 그래서 MediaPipe의 palm detector +
hand landmark 두 TFLite 모델을 `react-native-fast-tflite`로 직접 구동하는 2단계 파이프라인을
자체 구현했다. 로직은 MediaPipe 공식 C++ 계산기 소스(`ssd_anchors_calculator.cc`,
`tensors_to_detections_calculator.cc`, `detections_to_rects_calculator.cc`,
`rect_transformation_calculator.cc`)를 그대로 포팅한 것이다.

## 관련 파일

| 파일 | 역할 |
| :--- | :--- |
| `src/utils/analyzeHandImage.ts` | 파이프라인 진입점. `photoUri` → `Point3D[] \| null` |
| `src/utils/mediapipe/palmDetector.ts` | 1단계: palm 검출 (SSD anchor 디코딩 포함) |
| `src/utils/mediapipe/anchors.ts` | SSD anchor 생성 (24×24×2 + 12×12×6 = 2016개) |
| `src/utils/mediapipe/handRoi.ts` | palm 검출 결과 → 회전 정렬된 정사각 ROI 계산 (scale 2.6x, shift_y -0.5) |
| `src/utils/mediapipe/tensorRender.ts` | Skia 기반 이미지 디코드 및 회전 크롭 → 모델 입력 텐서 변환 |
| `src/utils/mediapipe/tensorUtils.ts` | 텐서 shape/dtype 관련 유틸 |
| `src/utils/tfliteModel.ts` | 모델 로딩 ([[Release_Build_Asset_Loading]] 참고) |

## 처리 순서

1. **팜 검출** (`detectPalm`): 사진 전체를 팜 검출 모델에 넣어 손바닥 위치와 정렬용
   키포인트(손목, 중지 MCP)를 찾는다. SSD anchor(2016개) 기반으로 박스/키포인트를
   디코딩한다. 실패하면 `null` 반환 → [[Hand_Not_Detected_Modal]].
2. **ROI 계산** (`computeHandRoi`): 손목→중지 MCP 벡터가 90°(수직)가 되도록 회전각을
   구하고, 정사각형으로 확장된 회전 크롭 영역(ROI)을 계산한다.
3. **회전 크롭 + 랜드마크 추론**: ROI를 Skia로 회전/크롭해 `hand_landmark_lite` 모델에
   넣는다. 출력은 `[landmarks(63), presence(1), handedness(1), world_landmarks(63)]`
   순서이며, `presence` 점수가 임계값(0.5) 미만이면 손이 없는 것으로 간주한다.
4. **좌표 역변환**: 모델 출력은 회전 정렬된 crop 좌표계 기준이므로,
   `inverseRotatedPoint()`로 원본 사진 전체 기준 정규화 좌표(0.0~1.0)로 되돌린다.
   `ResultScreen`은 항상 원본 사진 위에 그리기 때문이다.

```typescript
// 실제 사용 시그니처 (src/utils/analyzeHandImage.ts)
export const analyzeHandImage = async (uri: string): Promise<Point3D[] | null> => { ... };
```

## 손이 여러 개 인식된 경우
palm 검출 단계에서 신뢰도가 가장 높은 박스 하나만 채택하도록 구현되어 있어, 자연스럽게
첫 번째(가장 확실한) 손만 사용된다.
