---
title: 보건교육용 Germ Snap(세균 카메라) Docs
---

# 🧼 보건교육용 Germ Snap (세균 카메라)

초등학교 보건교육용 손 세균 시각화 앱 "Germ Snap"(한국어명: 세균 카메라)의
기획 및 개발 명세서입니다.
Phase 1~5 초기 구현 이후에도 실기기 테스트를 거쳐 계속 개정되는 문서이니, 코드를
수정하기 전 최신 내용을 먼저 확인하세요.

## 📌 허브 문서 (개요, 여기서부터 시작)
- [[00_Overview|00. 프로젝트 개요]]
- [[01_Architecture_Flow|01. 시스템 구조 및 화면 흐름]]
- [[02_Camera_and_AI|02. 카메라 및 AI 손 인식]]
- [[03_Germ_and_Clean_Rendering|03. 세균 렌더링 및 인터랙션]]
- [[04_Asset_and_Development_Guide|04. 에셋 및 개발 가이드]]

## 🗂️ 세부 노드 (`notes/`)
각 허브 문서 하단의 링크를 따라가면 아래처럼 주제별 세부 노드로 이어진다. 그래프
뷰로 보면 허브를 중심으로 관련 노드들이 방사형으로, 서로 관련된 노드끼리는
가로로도 연결돼 있다.

- **카메라/AI**: [[Two_Stage_Hand_Pipeline]] · [[Release_Build_Asset_Loading]] · [[Camera_Capture_and_Crop]] · [[Hand_Not_Detected_Modal]]
- **세균/이펙트 렌더링**: [[Rendering_Architecture]] · [[Germ_Display_Mode]] · [[Germ_Generation_Algorithm]] · [[Hand_Silhouette_Sampling]] · [[Germ_Animations]] · [[Microscope_Haze_Layer]] · [[After_Mode_Effects]] · [[Pinch_Zoom_Gesture]]
- **에셋/설정**: [[Dummy_Component_Fallback]] · [[Asset_Folder_Structure]] · [[Settings_and_Persistence]] · [[Localization]] · [[Handwash_Video]]
- **배포**: [[Privacy_Policy]] · [[Store_Listing]]
