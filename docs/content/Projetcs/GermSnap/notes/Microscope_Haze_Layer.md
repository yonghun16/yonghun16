# 현미경 뿌연 조명 효과 (Haze Layer)

## 🔗 상위 문서
[[03_Germ_and_Clean_Rendering]]

## 관련 노드
[[Germ_Display_Mode]] · [[Rendering_Architecture]]

---

## 문제: 사진과 세균의 해상도 이질감
고배율로 확대하면 사진(피부)은 원본 카메라 해상도의 한계로 깨져 보이는데, 세균은
별도의 고화질 이미지라 계속 또렷해서 이질감이 생긴다 ([[Rendering_Architecture|Skia
내부 transform으로 확대해도]] 사진 자체의 원본 픽셀 수 한계는 어쩔 수 없다).

## 해결: 반투명 레이어로 가리기
사진 위·세균 아래에 반투명 레이어를 깔아 실제 현미경 조명처럼 뿌옇게 가린다.

- **색상**: 순백색이 아니라 **주광색**(`#EAF4FF`, 약 6500K로 살짝 푸른 기가 도는
  흰색). 실제 현미경 백라이트가 주광색에 가깝다는 피드백에 따라 흰색에서 바꿨다.
- **불투명도**: `scale`에 따라 `0 → 0.55`로 `interpolate`된다 (핀치 확대할수록
  뿌옇게).

## [[Germ_Display_Mode|캐릭터 모드]]에서는 그리지 않는다
캐릭터 모드는 사실적인 사진이 아니라 벡터 도형(`DummyGerm`)을 쓰고 최대 배율도
4배로 낮아서, 애초에 이 haze 레이어가 가리려는 "사진-세균 해상도 이질감" 자체가
발생하지 않는다.

```typescript
{germDisplayMode === 'MICROSCOPE' && (
  <Rect x={displayRect.x} y={displayRect.y} width={displayRect.width} height={displayRect.height}
        color={HAZE_COLOR} opacity={hazeOpacity} />
)}
```
