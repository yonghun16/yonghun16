---
title: Quartz
description: Quartz 기반 옵시디언 웹 배포 가이드 (설치 성공 시나리오 반영)
tags:
  - Quartz
  - 가이드
  - 체크리스트
---

# Quartz

루트 프로젝트 내부의 `docs/` 디렉토리에 **Quartz**를 세팅하고, GitHub Pages로 자동 배포하는 구성 가이드입니다. 실제로 설치에 성공한 시나리오를 기준으로 정리했습니다.

## 1. 프로젝트 생성 및 Git 초기화

```bash
# 루트 프로젝트 디렉토리 생성 및 이동
mkdir my-project
cd my-project

# Git 저장소 초기화 및 첫 커밋
git init -b main
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"

# 원격 저장소 연결 및 푸시
git remote add origin https://github.com/사용자이름/my-project.git
git push -u origin main
```

## 2. `docs/` 디렉토리에 Quartz 설치

```bash
# Quartz 저장소를 docs 폴더로 클론
git clone https://github.com/jackyzha0/quartz.git docs
cd docs

# 중요: docs 내부의 .git을 삭제하여 서브모듈 충돌 방지
rm -rf .git

# 의존성 설치 및 초기화
npm install --legacy-peer-deps
npx quartz create
```

초기화 프롬프트 선택:

- `Empty Quartz` 선택
- `Treat title as page title` 선택

> Quartz 빌더는 기본적으로 `content/` 내부의 마크다운을 렌더링하므로, 문서는 항상 `docs/content/`에 위치합니다.

## 3. 사이트 기본 설정 (`docs/quartz.config.yaml`)

`npx quartz create`로 생성되는 설정 파일은 YAML입니다. `docs/quartz.config.yaml` 파일을 열어 상단 타이틀과 배포 주소를 수정합니다.

```yaml
# yaml-language-server: $schema=./quartz/plugins/quartz-plugins.schema.json
configuration:
  pageTitle: 나만의 프로젝트 타이틀 # 좌측 상단에 표시될 사이트 제목
  pageTitleSuffix: ""
  enableSPA: true
  enablePopovers: true
  analytics:
    provider: plausible
  locale: ko-KR                    # 한국어 설정
  baseUrl: 사용자이름.github.io/my-project
  ignorePatterns:
    - private
    - templates
    - .obsidian
```

- **`baseUrl`**: `https://`와 끝의 `/`를 제외한 순수 경로 (`<아이디>.github.io/<레포명>`)
- **`locale`**: `ko-KR` (한글 표기 지원)
- **`ignorePatterns`**: `private`, `templates`, `.obsidian`처럼 퍼블리시하지 않을 폴더 지정

나머지 플러그인/레이아웃 설정(폴더 페이지, 태그 페이지, 그래프, 백링크 등)도 같은 파일 안에 이어서 정의되어 있으므로, 새 사이트를 세팅할 때는 이 파일 전체를 기준으로 참고합니다.

## 4. GitHub Actions 자동 배포 워크플로우 설정

루트 디렉토리(`my-project`)로 이동한 뒤 워크플로우 파일을 생성합니다.

```bash
cd ..
mkdir -p .github/workflows
```

`.github/workflows/deploy.yml`:

```yaml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - main # 기본 브랜치가 master면 master로 수정

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      # --legacy-peer-deps 플래그를 추가하여 패키지 버전을 무시하고 설치
      - name: Install Dependencies
        run: |
          cd docs
          npm install --legacy-peer-deps

      # Quartz 빌드 실행
      - name: Build Quartz
        run: |
          cd docs
          npx quartz build

      # 결과물 업로드
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: docs/public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 5. GitHub 저장소 Pages 설정 및 첫 배포

1. 전체 소스 코드 커밋 및 푸시:

```bash
git add .
git commit -m "feat: setup quartz in docs and update github actions"
git push origin main
```

2. GitHub 저장소 웹페이지 → **Settings** → 좌측 **Pages** 클릭
3. **Build and deployment**의 **Source**를 `GitHub Actions`로 선택

## 6. 메인 접속 페이지 `content/index.md` (중요 ⭐️)

Quartz는 `content` 폴더 내부의 `index.md`를 웹사이트 대문(`index.html`)으로 사용합니다. 이 파일이 없으면 웹 페이지 대신 **RSS 피드 XML**이 출력되므로 반드시 만들어야 합니다.

- 파일 경로: `docs/content/index.md`

```markdown
---
title: <프로젝트명> Docs
---

# <프로젝트명>

프로젝트 소개 문장.

## 주요 문서 목록
- [[문서명|표시할 제목]]
```

## 7. 일상적인 개발 & 문서 작성 루틴

- **문서 작성 위치:** 모든 마크다운(`.md`) 파일은 `docs/content/` 폴더 내에 작성합니다.
- **로컬 실시간 미리보기:**

```bash
cd docs
npx quartz build --serve
```

- **수정사항 배포:**

```bash
git add docs/
git commit -m "docs: 문서 업데이트"
git push origin main
```

## 8. 주요 오류 및 해결 방법 (Troubleshooting)

| 증상 / 오류 메시지 | 원인 | 해결 방법 |
| --- | --- | --- |
| **404 Not Found** | GitHub Pages 설정 미비 또는 `baseUrl` 오타 | 1. Settings → Pages Source를 `GitHub Actions`로 변경<br>2. `quartz.config.yaml`의 `baseUrl` 확인 |
| **`This XML file does not appear...` (RSS 피드 출력)** | `content/index.md` 파일 누락 | `content/index.md` 대문 파일 생성 후 재푸시 |
| **스타일 깨짐 (CSS 미적용)** | `baseUrl`에 `https://`가 포함됨 | `baseUrl`에서 `https://` 및 끝의 `/` 제거 |
| **`.git` 서브모듈 충돌** | `git clone`으로 받은 `docs/`에 `.git`이 남아있음 | `docs/` 안의 `.git`을 삭제(`rm -rf docs/.git`) 후 루트에서 다시 커밋 |
