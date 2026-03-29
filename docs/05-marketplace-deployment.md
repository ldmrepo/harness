# Part 5 — 마켓플레이스 등록 및 배포

> [← 인덱스로 돌아가기](../README.md) | [← Part 4](04-meta-skill-implementation.md)

---

## 5.1 플러그인 vs 스킬

하네스를 배포하는 두 가지 방법.

| 방식 | 구조 | 장점 | 단점 |
|------|------|------|------|
| **글로벌 스킬** | `~/.claude/skills/harness/` | 즉시 사용, 설정 불필요 | 개인 사용, 배포 불가 |
| **플러그인 + 마켓플레이스** | GitHub + `plugin.json` | 공유/배포 가능, 버전 관리 | GitHub 레포 필요 |

**플러그인 = 스킬의 배포 패키징 단위.** 스킬 파일은 동일하고 `plugin.json`과 `marketplace.json`이 추가된다.

---

## 5.2 플러그인 디렉토리 구조

Claude Code 플러그인의 표준 구조.

```
harness-plugin/                    ← GitHub 레포지토리 루트
├── .claude-plugin/
│   ├── plugin.json                ← 플러그인 메타데이터 (필수)
│   └── marketplace.json           ← 마켓플레이스 카탈로그
├── skills/
│   └── harness/
│       ├── SKILL.md               ← 메타 스킬 진입점
│       └── references/
│           ├── agent-design-patterns.md
│           ├── skill-writing-guide.md
│           ├── orchestrator-template.md
│           ├── team-examples.md
│           └── qa-agent-guide.md
└── README.md
```

**중요 규칙:**
- `commands/`, `agents/`, `skills/`, `hooks/` 는 반드시 **레포 루트 레벨**에 위치
- `.claude-plugin/` 안에는 `plugin.json`만
- Claude Code가 표준 디렉토리를 자동 탐색

---

## 5.3 plugin.json

플러그인 메타데이터. `name` 하나만 필수.

```json
{
  "name": "harness",
  "version": "1.0.0",
  "description": "A meta-skill that designs domain-specific agent teams,
    defines specialized agents, and generates the skills they use.",
  "author": {
    "name": "Your Name",
    "email": "your@email.com",
    "url": "https://github.com/your-username"
  },
  "homepage": "https://github.com/your-username/harness",
  "repository": "https://github.com/your-username/harness",
  "license": "Apache-2.0",
  "keywords": [
    "harness",
    "agent-team",
    "multi-agent",
    "skill-architect",
    "orchestration"
  ],
  "skills": "./skills/"
}
```

**버전 관리:** `version` 변경 없이 코드만 바꾸면 기존 사용자에게 업데이트가 전달되지 않는다. 변경 시 반드시 버전 bump.

---

## 5.4 marketplace.json

마켓플레이스 카탈로그. `/plugin marketplace add` 명령으로 등록되는 파일.

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "harness",
  "description": "Agent Team & Skill Architect plugin for Claude Code.",
  "owner": {
    "name": "Your Name",
    "email": "your@email.com"
  },
  "plugins": [
    {
      "name": "harness",
      "description": "A meta-skill that designs domain-specific agent teams...",
      "version": "1.0.0",
      "author": {
        "name": "Your Name",
        "email": "your@email.com"
      },
      "source": "./",
      "category": "productivity",
      "keywords": ["harness", "agent-team", "multi-agent", "orchestration"],
      "homepage": "https://github.com/your-username/harness"
    }
  ]
}
```

**`source` 필드:** 플러그인 디렉토리 위치. `./`는 레포 루트 = 플러그인 자체가 레포 루트에 있음을 의미. 하나의 레포에 여러 플러그인이 있다면 `./plugins/harness`처럼 지정.

---

## 5.5 GitHub 배포 단계

### Step 1. GitHub 레포지토리 생성

```bash
cd harness-plugin
git init
git add .
git commit -m "feat: initial harness plugin v1.0.0"

# GitHub에 새 레포 생성 후
git remote add origin https://github.com/{username}/harness.git
git push -u origin main
```

### Step 2. plugin.json과 marketplace.json 업데이트

`your-username`을 실제 GitHub 사용자명으로 교체:

```json
"repository": "https://github.com/ldmrepo/harness",
"homepage": "https://github.com/ldmrepo/harness"
```

### Step 3. 마켓플레이스 등록 (Claude Code에서)

```
/plugin marketplace add ldmrepo/harness
```

성공 메시지:
```
✓ Successfully added marketplace: harness
```

### Step 4. 플러그인 설치

```
/plugin install harness@harness
```

### Step 5. 설치 확인

```
/plugin list
```

---

## 5.6 설치 방법 세 가지 비교

### 방법 1: 마켓플레이스 (권장, 공유 가능)

```bash
# Claude Code 내부에서
/plugin marketplace add ldmrepo/harness
/plugin install harness@harness
```

### 방법 2: 글로벌 스킬 (즉시 사용, 개인용)

```bash
cp -r skills/harness ~/.claude/skills/harness
```

### 방법 3: 프로젝트 로컬 스킬

```bash
mkdir -p .claude/skills
cp -r skills/harness .claude/skills/harness
```

---

## 5.7 플러그인 관리 명령어

```bash
# 마켓플레이스 관리
/plugin marketplace add {owner/repo}      # 마켓플레이스 등록
/plugin marketplace list                  # 등록된 마켓플레이스 목록
/plugin marketplace update                # 마켓플레이스 카탈로그 갱신
/plugin marketplace remove {name}         # 마켓플레이스 제거

# 플러그인 관리
/plugin install {name}@{marketplace}      # 설치
/plugin list                              # 설치된 플러그인 목록
/plugin disable {name}                    # 비활성화
/plugin enable {name}                     # 활성화
/plugin update {name}                     # 업데이트

# 플러그인 매니저 UI
/plugin                                   # 탭 UI 열기
                                          # Discover / Installed / Updates / Errors
```

---

## 5.8 자동 업데이트 설정

```bash
# 특정 마켓플레이스 자동 업데이트 활성화 (Claude Code UI에서)
/plugin → 마켓플레이스 선택 → Enable auto-update

# 모든 자동 업데이트 비활성화
export DISABLE_AUTOUPDATER=true

# Claude Code 업데이트는 끄되 플러그인만 자동 업데이트
export DISABLE_AUTOUPDATER=true
export FORCE_AUTOUPDATE_PLUGINS=true
```

업데이트 후 적용:
```
/reload-plugins
```

---

## 5.9 마켓플레이스 디렉토리

공개 플러그인을 찾을 수 있는 곳.

| 디렉토리 | URL | 특징 |
|----------|-----|------|
| **Anthropic 공식** | https://claude.com/plugins | 공식 검증, 자동 탑재 |
| **claudemarketplaces.com** | https://claudemarketplaces.com | 95+ 마켓플레이스 큐레이션 |
| **buildwithclaude.com** | https://buildwithclaude.com | 494+ 플러그인/스킬 |
| **GitHub (공식 레포)** | github.com/anthropics/claude-plugins-official | Anthropic 관리 외부 플러그인 |

### Anthropic 공식 마켓플레이스에 등록하려면

1. `anthropics/claude-plugins-official` 레포의 제출 폼 사용
2. 품질 및 보안 기준 충족 필요
3. 외부 플러그인은 `/external_plugins` 디렉토리에 등록

---

## 5.10 요구사항 및 제약

### 필수 요구사항

```bash
# 에이전트 팀 기능 활성화 (harness 사용 시)
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### 플러그인 파일 경로 제약

```json
// 올바른 경로 (상대 경로, ./ 시작)
"skills": "./skills/"
"commands": "./commands/"

// 잘못된 경로
"skills": "skills/"           // ./ 없음
"skills": "/Users/me/skills/" // 절대 경로
"skills": "../shared/"        // 경로 순회
```

### 파일명 규칙

- 모든 디렉토리/파일명: kebab-case (소문자 + 하이픈)
- `plugin.json`의 `name`: 소문자, 하이픈만, 64자 이하
- `SKILL.md` name 필드: 동일 규칙

---

## 5.11 트러블슈팅

| 문제 | 원인 | 해결 |
|------|------|------|
| `/plugin` 명령 없음 | zsh에서 실행 | `claude` 명령으로 Claude Code 진입 후 실행 |
| `command not found: openclaw` | .zshrc 경로 오류 | `npm install -g @anthropic-ai/claude-code` 재설치 |
| 마켓플레이스 추가 실패 | GitHub 레포 없음 또는 `marketplace.json` 없음 | 레포 생성 및 파일 확인 |
| 스킬 트리거 안 됨 | description 불명확 | 트리거 키워드 구체화, "pushy"하게 작성 |
| 에이전트 파일 로드 안 됨 | `.claude-plugin/` 안에 위치 | 레포 루트로 이동 |

---

## 5.12 전체 플로우 요약

```
1. 개발
   skills/harness/ 파일 작성
   └── SKILL.md (6 Phase)
   └── references/ (5개 파일)

2. 패키징
   .claude-plugin/plugin.json 생성
   .claude-plugin/marketplace.json 생성

3. 배포
   GitHub 레포 생성 → 파일 푸시

4. 등록 (Claude Code 내부)
   /plugin marketplace add {username}/harness
   /plugin install harness@harness

5. 사용
   "하네스 구성해줘"
   → 자동으로 .claude/agents/ + .claude/skills/ 생성
```

---

## 참고 링크

- Claude Code 플러그인 공식 문서: https://code.claude.com/docs/en/plugins-reference
- 마켓플레이스 공식 문서: https://code.claude.com/docs/en/discover-plugins
- Anthropic 공식 플러그인 레포: https://github.com/anthropics/claude-plugins-official
- 공식 플러그인 디렉토리: https://claude.com/plugins

> [← Part 4: 메타 스킬 구현](04-meta-skill-implementation.md) | [← 인덱스로 돌아가기](../README.md)
