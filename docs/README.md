# Harness Engineering 완전 가이드

> Claude Code 에이전트 팀 & 스킬 아키텍트 — 이론부터 실전 서비스 구축까지

---

## 문서 구성

이 문서는 Harness Engineering의 개념 이해부터 실제 메타 스킬 구축, 마켓플레이스 배포, 실전 서비스 적용까지 전 과정을 다룹니다.

```
harness-docs/
├── README.md                           ← 이 파일 (인덱스)
└── parts/
    ├── 01-harness-engineering.md       ← Part 1: Harness Engineering 이론
    ├── 02-claude-code-skills.md        ← Part 2: Claude Code 스킬 이해
    ├── 03-meta-skill-architecture.md   ← Part 3: 하네스 메타 스킬 아키텍처
    ├── 04-meta-skill-implementation.md ← Part 4: 메타 스킬 구현
    ├── 05-marketplace-deployment.md    ← Part 5: 마켓플레이스 등록 및 배포
    └── 06-real-world-usage.md          ← Part 6: 실전 서비스 구축 (문제은행)
```

---

## 빠른 시작

### 사전 요구사항

```bash
# Claude Code 설치 확인
which claude
# 없으면: npm install -g @anthropic-ai/claude-code

# 에이전트 팀 기능 활성화 (필수)
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### 설치

```bash
# 방법 1: 마켓플레이스를 통한 설치 (Claude Code 내부)
/plugin marketplace add {github-username}/harness
/plugin install harness@harness

# 방법 2: 글로벌 스킬로 직접 설치
cp -r skills/harness ~/.claude/skills/harness

# 설치 확인
/plugin list
```

### 사용

```
# 기본 트리거
하네스 구성해줘

# 실전 예시 — 문제은행 서비스
문제은행 서비스 구축 하네스를 구성해줘.
목표: 2022 개정 교육과정 기반 AI 문항 자동 생성 및 관리
```

---

## 파트별 요약

### [Part 1 — Harness Engineering 이론](parts/01-harness-engineering.md)

OpenAI가 2026년 2월 공개한 Harness Engineering 방법론의 핵심 개념과 커뮤니티 반응을 정리합니다.

- Harness Engineering이란 무엇인가
- Context Engineering vs Harness Engineering 차이
- Big Model vs Big Harness 논쟁
- 실증 데이터 (OpenAI 100만 줄, LangChain +13.7%, Harness A/B +60%)
- 산업 사례 (OpenAI, Stripe, Anthropic)
- IOSYS 프로젝트 적용 관점

### [Part 2 — Claude Code 스킬 이해](parts/02-claude-code-skills.md)

Claude Code Agent Skills의 구조, 작동 원리, 작성 방법을 설명합니다.

- 스킬이란 무엇인가 (절차적 지식 캡슐화)
- Progressive Disclosure 3단계 로딩 구조
- SKILL.md 구조와 작성 규칙
- description 작성 전략 (3인칭, 트리거 키워드)
- 자유도 설정 (높음/중간/낮음)
- 피드백 루프 패턴
- 스킬 개발 이터레이션 방법 (Claude A → Claude B)

### [Part 3 — 하네스 메타 스킬 아키텍처](parts/03-meta-skill-architecture.md)

스킬이 스킬을 만드는 메타 스킬의 개념 설계와 6가지 에이전트 아키텍처 패턴을 다룹니다.

- 메타 스킬 개념 (3가지 레이어)
- 6가지 아키텍처 패턴 및 선택 플로우차트
- 에이전트 팀 vs 서브에이전트 모드
- 파일 디렉토리 설계
- IOSYS 도메인별 패턴 매핑 (지식 그래프/문항은행/음성채점)

### [Part 4 — 메타 스킬 구현](parts/04-meta-skill-implementation.md)

실제 harness 메타 스킬의 파일별 구현 내용을 상세히 설명합니다.

- SKILL.md 6 Phase 워크플로우 전체 구조
- references/agent-design-patterns.md
- references/skill-writing-guide.md
- references/orchestrator-template.md (에이전트 간 데이터 계약 형식 포함)
- references/team-examples.md (IOSYS 5개 팀 예시)
- references/qa-agent-guide.md (2단계 검증 방법론)
- 실제 사용 흐름 예시

### [Part 5 — 마켓플레이스 등록 및 배포](parts/05-marketplace-deployment.md)

Claude Code 플러그인 구조와 마켓플레이스 등록, 배포 전 과정을 다룹니다.

- 플러그인 구조 (plugin.json, marketplace.json)
- GitHub 레포지토리 설정 및 배포 4단계
- 마켓플레이스 등록 및 설치 명령어
- 플러그인 관리 명령어 전체 목록
- 마켓플레이스 디렉토리 (claude.com/plugins 외)
- 트러블슈팅 가이드

### [Part 6 — 실전 서비스 구축: 문제은행](parts/06-real-world-usage.md)

harness 플러그인 설치부터 문제은행 서비스 구축까지 실전 전 과정을 안내합니다.

- 설치 확인 및 환경 설정
- 문제은행 하네스 트리거 프롬프트
- 자동 생성되는 에이전트/스킬 파일 구조
- 생성-검증 파이프라인 실행 흐름
- 생성 후 필수 검토 및 커스터마이징 포인트
- 실제 문항 생성 실행 예시

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
      ↓
Domain Service            ← 문제은행, 음성채점, 지식 그래프 등
```

---

## 학습 경로 추천

| 목적 | 추천 경로 |
|------|-----------|
| 개념만 빠르게 이해 | Part 1 → Part 3 요약 |
| 스킬 직접 작성 | Part 2 → Part 4 |
| 지금 바로 사용 | 빠른 시작 → Part 6 |
| 팀 배포/공유 | Part 5 |
| 전체 이해 | Part 1 → 2 → 3 → 4 → 5 → 6 순서대로 |

---

## 참고 자료

| 자료 | 링크 |
|------|------|
| OpenAI Harness Engineering 원문 | https://openai.com/index/harness-engineering/ |
| revfactory/harness GitHub | https://github.com/revfactory/harness |
| revfactory/harness-100 (100개 팀 예시) | https://github.com/revfactory/harness-100 |
| Claude Code 공식 스킬 문서 | https://code.claude.com/docs/en/skills |
| Claude Code 플러그인 레퍼런스 | https://code.claude.com/docs/en/plugins-reference |
| Claude Code 마켓플레이스 탐색 | https://claude.com/plugins |
| Anthropic 공식 플러그인 레포 | https://github.com/anthropics/claude-plugins-official |
| Martin Fowler 분석 | https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html |

---

*최종 업데이트: 2026년 3월*