---
title: LeanSpec 설치 가이드
description: 기획서/계획서(Spec)를 AI와 함께 작고 최신 상태로 관리하는 SDD CLI 도구 설치 및 연동 가이드
tags:
  - 가이드
  - 체크리스트
  - AI
---

# LeanSpec 설치 가이드

기획서/계획서(Spec)를 AI와 함께 작고 최신 상태로 관리하기 위한 Spec-Driven
Development(SDD) 도구. markmap처럼 구조를 시각화하는 대신, **문서가 애초에
복잡해지지 않도록** 작은 단위로 쪼개고 상태를 추적하는 방식으로 문제를 푼다.

## 1. CLI 설치

```bash
npm install -g @leanspec/cli
```

### 설치 후 EACCES 에러가 나는 경우

```
Failed to start leanspec: spawn .../\@leanspec/cli-darwin-arm64/leanspec EACCES
```

플랫폼별 바이너리 패키지가 실행 권한 없이 깔리는 경우 발생. 해결:

```bash
chmod +x $(npm root -g)/@leanspec/cli/node_modules/@leanspec/cli-darwin-arm64/leanspec
```

(경로는 OS/아키텍처에 따라 `cli-darwin-arm64` 부분이 다를 수 있음 — 에러 메시지에
찍힌 실제 경로를 그대로 사용)

그래도 안 되면 재설치:

```bash
npm uninstall -g @leanspec/cli
npm cache clean --force
npm install -g @leanspec/cli
```

## 2. 초기화

스펙을 관리할 프로젝트 폴더(예: `docs`)에서:

```bash
cd docs
leanspec init
```

대화형으로 몇 가지를 묻는다 (프로젝트명, draft 상태 사용 여부 등). 완료되면
아래 구조가 생성됨:

```
docs/
├── specs/                  ← 스펙 문서가 여기 쌓임
│   └── README.md
├── .lean-spec/
│   ├── config.json         ← 설정
│   ├── schemas/            ← 커스텀 스펙 스키마
│   └── templates/          ← 스펙 작성 템플릿
└── AGENTS.md                ← LeanSpec이 자동 생성하는 AI 작업 규칙
```

> ⚠️ `leanspec`과 `lean-spec` 둘 다 같은 바이너리를 가리킨다 (동일하게 동작).

## 3. 기본 명령어

```bash
leanspec create my-feature          # 새 스펙 생성 (frontmatter 자동 생성, 번호 자동 부여)
leanspec list                       # 스펙 목록
leanspec board                      # 칸반 보드 뷰
leanspec view <spec>                # 스펙 내용 보기
leanspec search "query"             # 스펙 검색
leanspec update <spec> --status <status>   # 상태 변경 (planned/in-progress/complete)
leanspec archive <spec>             # 보관 처리
leanspec ui                         # 웹 UI (localhost:3000)
```

## 4. AI 도구와 연동

### CLI 방식 (설정 없이 바로 사용 가능 — 추천 시작점)

AI 코딩 도구(Claude Code 등)가 위 명령어를 직접 bash로 호출하는 방식. 별도
설정 불필요, 지금 당장 시작 가능.

### MCP 방식 (선택, 나중에 전환 가능)

AI가 구조화된 데이터로 직접 받는 방식. `.mcp.json`에 등록:

```json
{
  "mcpServers": {
    "leanspec": {
      "command": "npx",
      "args": ["@leanspec/mcp"]
    }
  }
}
```

CLI로 충분히 잘 굴러가면 굳이 전환할 필요 없음. 스펙 개수가 많아져서
`search`/`deps` 등을 자주 쓰게 될 때 고려.

## 5. AGENTS.md 통합

`leanspec init`이 자동 생성하는 `AGENTS.md`는 LeanSpec 전용 규칙만 담고
있음. 프로젝트에 이미 다른 AI 작업 규칙(`CLAUDE.md` 등, 예: Obsidian/[[Quartz|Quartz]]
문서 관리 규칙)이 있다면 **하나의 `AGENTS.md`로 통합**할 것을 권장.

- 파일명은 반드시 `AGENTS.md`로 유지 — `leanspec` 관련 명령을 다시 실행하면
  이 파일명으로 규칙을 재생성할 수 있기 때문에, `CLAUDE.md` 등 다른 이름으로
  바꾸면 다음에 또 파일이 갈라질 수 있음
- 통합 시 시스템별로 섹션을 명확히 분리 (예: "Spec 관리(LeanSpec)" /
  "의사결정 기록(ADR)" / "일반 문서 관리(Vault)") — 각 시스템은 서로 다른
  frontmatter 스키마와 편집 방식을 가지므로 섞어 쓰지 않도록 안내

## 6. 실행 전 스모크 테스트 (버전별로 꼭 해볼 것)

LeanSpec은 활발히 개발 중이라(Rust 기반 "adapter API"로 아키텍처 마이그레이션
진행 중), 버전에 따라 일부 명령어가 스텁 상태일 수 있음. 설치 직후 아래
명령어들을 한 번씩 실행해서 실제로 동작하는지 확인할 것:

```bash
leanspec create test-spec
leanspec list
leanspec board
leanspec view test-spec
leanspec search test
leanspec update test-spec --status in-progress
leanspec validate
leanspec tokens test-spec
leanspec deps test-spec
leanspec rel add test-spec --depends-on other-spec
leanspec archive test-spec
```

### 알려진 이슈 (2026-09-05 기준, `@leanspec/cli@0.3.0`)

`validate`, `tokens`, `deps`, `analyze`, `rel`이 전부
`Error: '<cmd>' is not yet migrated to the adapter API`로 실패함.
`create`/`update`/`list`/`board`/`view`/`search`/`archive`는 정상 동작.

**원인**: LeanSpec 팀이 `list` 명령어를 먼저 마이그레이션(레퍼런스 구현)했고,
나머지 명령어는 다음 순번 마이그레이션 대상이라 일부러 스텁 에러를 내도록
처리해둔 상태. 우리 쪽 설정 문제가 아니라 라이브러리 자체의 과도기적 상태.

**대응**:

- CRUD(`create`/`update`/`list`/`board`/`view`/`search`/`archive`)만으로도
  "기획서를 짧게 쪼개서 관리한다"는 핵심 목적은 달성 가능 → 지금은 이것만 사용
- `validate` 등에 의존하는 워크플로우 규칙(AGENTS.md의 "완료 전 validate
  필수" 등)은 현재 버전에서 강제 불가 — AGENTS.md에 임시 경고 문구를 남겨두고
  CLI 업데이트 후 재확인
- 스모크 테스트 결과가 바뀌면(즉 위 명령어들이 정상화되면) AGENTS.md의
  경고 문구를 지우고 해당 규칙을 다시 활성화

```bash
npm view @leanspec/cli version   # 최신 버전 확인용, 가끔 체크
```

## 7. 첫 스펙 만들어보기

```bash
leanspec create my-first-feature
leanspec board
```

`specs/` 폴더에 frontmatter가 채워진 md 파일이 생성되는지, `board`에서 상태가
보이는지 확인하면 정상 설치 완료.
