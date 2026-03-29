# Harness Engineering 완전 가이드

> Claude Code 에이전트 팀 & 스킬 아키텍트 — 이론부터 실전 배포까지

---

## 문서 구성

이 문서는 Harness Engineering의 개념 이해부터 실제 메타 스킬 구축, 마켓플레이스 배포까지 전 과정을 다룹니다.

```
harness-docs/
├── README.md                          ← 이 파일 (인덱스)
└── parts/
    ├── 01-harness-engineering.md      ← Part 1: Harness Engineering 이론
    ├── 02-claude-code-skills.md       ← Part 2: Claude Code 스킬 이해
    ├── 03-meta-skill-architecture.md  ← Part 3: 하네스 메타 스킬 아키텍처
    ├── 04-meta-skill-implementation.md ← Part 4: 메타 스킬 구현
    └── 05-marketplace-deployment.md   ← Part 5: 마켓플레이스 등록 및 배포
```

---

## 빠른 시작

```bash
# 글로벌 스킬로 직접 설치
cp -r skills/harness ~/.claude/skills/harness

# 또는 마켓플레이스를 통한 설치
/plugin marketplace add {github-username}/harness
/plugin install harness@harness
```

설치 후 Claude Code에서:
```
하네스 구성해줘
```

---

## 파트별 요약

### [Part 1 — Harness Engineering 이론](parts/01-harness-engineering.md)

OpenAI가 2026년 2월 공개한 Harness Engineering 방법론의 핵심 개념과 커뮤니티 반응을 정리합니다.

- Harness Engineering이란 무엇인가
- Context Engineering vs Harness Engineering 차이
- Big Model vs Big Harness 논쟁
- 실증 데이터 및 산업 사례
- IOSYS 프로젝트 적용 관점

### [Part 2 — Claude Code 스킬 이해](parts/02-claude-code-skills.md)

Claude Code Agent Skills의 구조, 작동 원리, 작성 방법을 설명합니다.

- 스킬이란 무엇인가 (절차적 지식 캡슐화)
- Progressive Disclosure 원리
- SKILL.md 구조와 작성 규칙
- description 작성 전략
- 자유도 설정 (높음/중간/낮음)
- 피드백 루프 패턴

### [Part 3 — 하네스 메타 스킬 아키텍처](parts/03-meta-skill-architecture.md)

스킬이 스킬을 만드는 메타 스킬의 개념 설계와 6가지 에이전트 아키텍처 패턴을 다룹니다.

- 메타 스킬 개념 (스킬 × 하네스)
- 6가지 아키텍처 패턴
- 에이전트 팀 vs 서브에이전트 모드
- 파일 디렉토리 설계
- IOSYS 도메인별 패턴 매핑

### [Part 4 — 메타 스킬 구현](parts/04-meta-skill-implementation.md)

실제 harness 메타 스킬의 파일별 구현 내용을 설명합니다.

- SKILL.md (6 Phase 워크플로우)
- references/agent-design-patterns.md
- references/skill-writing-guide.md
- references/orchestrator-template.md
- references/team-examples.md
- references/qa-agent-guide.md

### [Part 5 — 마켓플레이스 등록 및 배포](parts/05-marketplace-deployment.md)

Claude Code 플러그인 구조와 마켓플레이스 등록, 배포 전 과정을 다룹니다.

- 플러그인 구조 (plugin.json, marketplace.json)
- GitHub 레포지토리 설정
- 마켓플레이스 등록 방법
- 설치 및 검증
- 마켓플레이스 디렉토리 목록

---

## 핵심 개념 관계도

```
Prompt Engineering
      ↓
Context Engineering       ← 에이전트에게 올바른 정보를 주는 것
      ↓
Harness Engineering       ← 에이전트가 안정적으로 동작하는 환경 구축
      ↓
Meta Skill (Harness)      ← 하네스를 자동 생성하는 스킬
      ↓
Agent Team + Skills       ← 실제 산출물 (.claude/agents/, .claude/skills/)
```

---

## 참고 자료

| 자료 | 링크 |
|------|------|
| OpenAI Harness Engineering 원문 | https://openai.com/index/harness-engineering/ |
| revfactory/harness GitHub | https://github.com/revfactory/harness |
| Claude Code 공식 스킬 문서 | https://code.claude.com/docs/en/skills |
| Claude Code 플러그인 마켓플레이스 | https://claude.com/plugins |
| Anthropic 공식 플러그인 레포 | https://github.com/anthropics/claude-plugins-official |

---

*최종 업데이트: 2026년 3월*
