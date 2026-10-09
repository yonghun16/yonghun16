# 에셋 폴더 구조 (현재 상태)

## 🔗 상위 문서
[[04_Asset_and_Development_Guide]]

## 관련 노드
[[Dummy_Component_Fallback]] · [[Germ_Generation_Algorithm]] · [[Two_Stage_Hand_Pipeline]]

---

```text
assets/
 ├── icon.png                      # 앱 아이콘 (iOS + 기본), Home 화면에서도 사용
 ├── android-icon-foreground.png   # Android adaptive icon 전경
 ├── android-icon-background.png   # Android adaptive icon 배경
 ├── android-icon-monochrome.png   # Android 13+ 단색(테마) 아이콘
 ├── favicon.png / splash-icon.png
 ├── germs/                        # 실제 세균 사진 (PNG, 투명 배경) — 현미경 모드 전용
 │    ├── germ1.png ~ germ10.png   # 10종 전부 로드 후 촬영마다 6종 무작위 선택 (Germ_Generation_Algorithm)
 ├── models/                       # 온디바이스 손 인식용 TFLite 모델 (Two_Stage_Hand_Pipeline)
 │    ├── palm_detection_lite.tflite
 │    └── hand_landmark_lite.tflite
 ├── effects/                      # (미사용, .gitkeep만 존재) 반짝이 PNG 예정 자리
 └── sounds/                       # (미사용, .gitkeep만 존재) 효과음 mp3 예정 자리
```

`effects/`, `sounds/`는 아직 실제 파일이 없다. [[Germ_Display_Mode|캐릭터 모드]]의
세균과 AFTER 모드의 반짝이는 지금도 [[Dummy_Component_Fallback|벡터 도형]]만으로
완성도 있게 동작하므로, PNG 에셋은 "있으면 더 좋은" 선택 사항이지 필수가 아니다.

## 효과음 상태
`assets/sounds/`에 실제 mp3 파일이 없어 `src/utils/sound.ts`의
`playScanSound`/`playCleanSound`/`playErrorSound`는 현재 no-op이다. `expo-av`는
실기기에서 네이티브 크래시(`UnsatisfiedLinkError`, dlopen 실패)를 일으켜 제거했고,
mp3 에셋이 준비되면 Expo가 권장하는 `expo-audio`로 교체할 계획이다.
