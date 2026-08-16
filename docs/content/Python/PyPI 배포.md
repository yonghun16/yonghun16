---
title: PyPI 배포
description: vim-goban 기준 버전업부터 PyPI 업로드, 배포 확인까지 전체 흐름
tags:
  - Python
  - 가이드
  - 체크리스트
---

# PyPI 배포

`vim-goban` 패키지를 기준으로 한 PyPI 배포 절차입니다.

## 전체 흐름

```text
코드 수정
 ↓
버전 증가
 ↓
빌드 파일 삭제
 ↓
패키지 생성
 ↓
PyPI 업로드
 ↓
사용자는 pipx upgrade
```

## 1. 버전 올리기

`pyproject.toml`을 확인합니다.

```toml
[project]
name = "vim-goban"
version = "0.1.0"
```

버전 규칙 추천:

- 버그 수정 → `0.1.1`
- 기능 추가(캡처, 패스, AI 개선 등) → `0.2.0`

> UI 개선 + 게임 기능 추가처럼 규모가 있는 변경이면 `0.2.0`을 추천합니다.

## 2. 기존 빌드 삭제

이전 빌드 산출물을 반드시 지우고 새로 빌드합니다.

```bash
rm -rf dist build *.egg-info
# 또는
rm -rf dist/
rm -rf vim_goban.egg-info/
```

## 3. 다시 빌드

```bash
python -m build
```

> `No module named build` 에러가 나면 `build` 패키지가 설치되지 않은 것입니다. 현재 파이썬 버전(예: `python3.11`)에 맞춰 설치합니다.
>
> ```bash
> python3.11 -m pip install build
> # 또는 pipx로 설치했다면
> pyproject-build
> ```

## 4. 패키지 확인

빌드 결과:

```text
dist/
├── vim_goban-0.2.0-py3-none-any.whl
└── vim_goban-0.2.0.tar.gz
```

```bash
ls dist
```

## 5. PyPI 업로드

```bash
twine upload dist/*
```

토큰을 입력하고 업로드가 성공하면 다음처럼 출력됩니다.

```text
Uploading vim_goban-0.2.0-py3-none-any.whl
100%
View at:
https://pypi.org/project/vim-goban/
```

## 6. 설치 테스트

```bash
pipx uninstall vim-goban
pipx install vim-goban
goban
```

## 7. 기존 사용자 업데이트

```bash
pipx upgrade vim-goban
```

## 8. Git Tag (권장)

PyPI로 배포하는 프로젝트라면 태그를 함께 남겨 GitHub Release 관리를 깔끔하게 합니다.

```bash
git add .
git commit -m "release v0.2.0"

git tag v0.2.0

git push origin main --tags
```

## 버전 관리 방향

`vim-goban`은 단순 실습 프로젝트 수준을 넘어 실제 CLI 패키지 형태가 되었으므로, 앞으로는 다음 기준으로 관리합니다.

```text
0.2.0  ← 현재
0.2.1  ← 버그 수정
0.3.0  ← 큰 기능
1.0.0  ← 안정 버전
```

다음 단계로 GitHub Actions를 붙여 `git tag → 자동 PyPI 배포`까지 연결하면 오픈소스 프로젝트 운영 형태에 가까워집니다.
