---
name: harness
description: >
  This skill should be used when the user wants to "build a harness",
  "design an agent team", "set up a harness", "configure an agent team for this project",
  or needs to generate agent definitions (.claude/agents/) and skills (.claude/skills/)
  tailored to a specific domain. Analyzes the project domain, selects an architecture
  pattern from 6 options, and generates a complete, production-ready agent team with
  orchestration, skills, and validation. Use when the user mentions harness, agent team,
  multi-agent system, or asks to automate a complex multi-step workflow with specialized agents.
---

# Harness — Agent Team & Skill Architect

도메인에 맞는 전문 에이전트 팀을 설계하고, 에이전트가 사용할 스킬까지 자동 생성하는 메타 스킬.

## 실행 전 확인 사항

에이전트 팀 기능이 활성화되어 있어야 합니다:
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

## 6 Phase 워크플로우

```
Phase 1: 도메인 분석
    ↓
Phase 2: 팀 아키텍처 설계 (패턴 선택)
    ↓
Phase 3: 에이전트 정의 생성 (.claude/agents/)
    ↓
Phase 4: 스킬 생성 (.claude/skills/)
    ↓
Phase 5: 통합 및 오케스트레이션
    ↓
Phase 6: 검증 및 테스트
```

진행 체크리스트를 복사해서 추적하라:

```
Harness 구성 진행 상황:
- [ ] Phase 1: 도메인 분석 완료
- [ ] Phase 2: 아키텍처 패턴 선택 완료
- [ ] Phase 3: 에이전트 정의 파일 생성 완료
- [ ] Phase 4: 스킬 파일 생성 완료
- [ ] Phase 5: 오케스트레이터 구성 완료
- [ ] Phase 6: 검증 테스트 통과
```

---

## Phase 1: 도메인 분석

사용자의 요청에서 다음을 파악한다:

**파악 항목:**
1. **핵심 목표** — 이 하네스가 달성해야 할 최종 결과물
2. **주요 작업 유형** — 순차적 / 병렬 / 조건부 / 반복적
3. **필요 전문성** — 각 에이전트가 담당할 도메인 (분석, 생성, 검증, 배포 등)
4. **데이터 흐름** — 에이전트 간 무엇이 전달되는가
5. **품질 기준** — 성공의 기준은 무엇인가
6. **외부 도구** — MCP 서버, API, 파일시스템 등

불명확한 항목이 있으면 사용자에게 질문한다. 분석 결과를 한국어로 요약해서 사용자에게 확인받는다.

---

## Phase 2: 팀 아키텍처 설계

아키텍처 패턴 선택 가이드 → **[references/agent-design-patterns.md](references/agent-design-patterns.md)**

**빠른 패턴 선택 기준:**

| 상황 | 권장 패턴 |
|------|-----------|
| A가 끝나야 B가 시작 | 파이프라인 |
| 독립 작업을 동시에 | 팬아웃/팬인 |
| 상황에 따라 다른 전문가 | 전문가 풀 |
| 만들고 → 검수 반복 | 생성-검증 |
| 중앙에서 동적 분배 | 감독자 |
| 큰 목표 → 작은 목표 재귀 | 계층적 위임 |

**실행 모드 결정:**

```
2개 이상 에이전트 + 협업 필요?
  → YES: 에이전트 팀 모드 (TeamCreate + SendMessage + TaskCreate)
  → NO:  서브 에이전트 모드 (Agent 도구 직접 호출)
```

설계 결과를 사용자에게 확인받은 뒤 Phase 3으로 진행한다.

---

## Phase 3: 에이전트 정의 생성

각 에이전트를 `.claude/agents/` 디렉토리에 마크다운 파일로 생성한다.

**에이전트 파일 템플릿:**

```markdown
---
name: {agent-name}
description: >
  This agent should be used when {구체적 트리거 조건}.
  Specializes in {전문 영역}. Use for {사용 시나리오}.
tools: [Read, Write, Bash, {필요한 도구들}]
---

# {에이전트 이름} — {역할 한 줄 요약}

## 역할
{이 에이전트가 담당하는 책임 범위}

## 입력
{이 에이전트가 받는 데이터/컨텍스트 형식}

## 출력
{이 에이전트가 생성하는 결과물 형식}

## 핵심 작업 흐름
1. {step 1}
2. {step 2}
3. ...

## 품질 기준
- {완료 조건 1}
- {완료 조건 2}

## 제약 사항
- {이 에이전트가 하지 말아야 할 것}
```

**에이전트 수 가이드라인:**
- 최소: 2개 (오케스트레이터 + 실행자)
- 권장: 3~5개 (전문화 vs 복잡도 균형)
- 최대: 7개 (그 이상은 오케스트레이션 복잡도 급증)

생성된 에이전트 목록과 역할을 사용자에게 확인받는다.

---

## Phase 4: 스킬 생성

스킬 작성 상세 가이드 → **[references/skill-writing-guide.md](references/skill-writing-guide.md)**

각 에이전트가 사용할 스킬을 `.claude/skills/` 에 생성한다.

**스킬 파일 구조:**

```
.claude/skills/
├── {skill-name}/
│   ├── SKILL.md          ← 진입점 (500줄 이하)
│   └── references/
│       ├── {domain}.md   ← 온디맨드 로드
│       └── examples.md   ← 온디맨드 로드
```

**스킬 분류:**
1. **도메인 스킬** — 특정 에이전트 전용 (예: `analyze-curriculum`, `generate-item`)
2. **공유 스킬** — 여러 에이전트가 공통 사용 (예: `validate-output`, `format-report`)
3. **오케스트레이터 스킬** — 팀 조율 전용 (예: `orchestrate-{team-name}`)

---

## Phase 5: 통합 및 오케스트레이션

오케스트레이터 템플릿 → **[references/orchestrator-template.md](references/orchestrator-template.md)**

**오케스트레이터 스킬 생성:**
- `.claude/skills/orchestrate-{team-name}/SKILL.md` 생성
- 에이전트 간 데이터 전달 형식 정의
- 에러 핸들링 프로토콜 포함
- 체크포인트 저장 로직 포함 (장기 실행 대비)

**에이전트 팀 모드 기본 호출 패턴:**
```
TeamCreate → 에이전트들 등록
  ↓
TaskCreate → 첫 번째 에이전트에게 작업 할당
  ↓
SendMessage → 에이전트 간 결과 전달
  ↓
결과 집계 및 최종 출력
```

---

## Phase 6: 검증 및 테스트

QA 에이전트 통합 가이드 → **[references/qa-agent-guide.md](references/qa-agent-guide.md)**

**검증 체크리스트:**

```
검증 항목:
- [ ] 각 에이전트 파일 YAML frontmatter 유효성 확인
- [ ] 스킬 description이 3인칭으로 작성되었는지 확인
- [ ] SKILL.md 본문이 500줄 이하인지 확인
- [ ] references/ 파일이 모두 존재하는지 확인
- [ ] 에이전트 간 데이터 형식이 일치하는지 확인
- [ ] 오케스트레이터 스킬 트리거 테스트
- [ ] 에러 케이스 처리 확인
```

**드라이런 테스트:**
1. 스킬 단독 실행 (`/skill-name` 으로 직접 호출)
2. With-skill vs Without-skill 비교 (같은 태스크를 스킬 있을 때/없을 때)
3. 실제 도메인 데이터로 통합 테스트

**실패 시 처리 원칙:**
- 에이전트가 어려움을 겪을 때 → "무엇이 빠져 있는가?"를 찾아서 레포에 추가
- 모델을 탓하지 말고 환경을 개선한다 (Harness Engineering 원칙)

---

## 산출물 구조 예시

```
프로젝트/
├── .claude/
│   ├── agents/
│   │   ├── orchestrator.md    ← 팀 총괄
│   │   ├── analyst.md         ← 분석 전담
│   │   ├── builder.md         ← 생성 전담
│   │   └── validator.md       ← 검증 전담
│   └── skills/
│       ├── orchestrate-{name}/
│       │   └── SKILL.md
│       ├── analyze/
│       │   ├── SKILL.md
│       │   └── references/
│       ├── build/
│       │   ├── SKILL.md
│       │   └── references/
│       └── validate/
│           ├── SKILL.md
│           └── references/
```

팀 구성 예시는 → **[references/team-examples.md](references/team-examples.md)**
