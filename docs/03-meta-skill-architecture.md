# Part 3 — 하네스 메타 스킬 아키텍처

> [← 인덱스로 돌아가기](../README.md) | [← Part 2](02-claude-code-skills.md)

---

## 3.1 메타 스킬 개념

**메타 스킬 = 스킬을 만드는 스킬**

하네스 메타 스킬은 두 가지 레이어를 통합한다.

```
Skill      = 절차적 지식 캡슐화 단위
Harness    = 에이전트 팀 전체를 통제하는 환경 인프라
─────────────────────────────────────────────────────
Meta Skill = Skill이 Harness 전체를 자동 생성하는 상위 레이어
```

### 세 가지 레이어

**레이어 1 — 메타 스킬 (Harness 자체)**
- `SKILL.md` + `references/` 6개 파일로 구성된 플러그인
- 사용자의 도메인을 분석해서 어떤 에이전트가 필요한지 판단
- "하네스 구성해줘" 한 마디로 트리거

**레이어 2 — 에이전트 레이어 (생성된 팀)**
- `.claude/agents/` 디렉토리에 마크다운 파일로 생성
- 역할, 전문 영역, 다른 에이전트와의 협업 방식 명세
- `analyst.md`, `builder.md`, `qa.md` 등

**레이어 3 — 스킬 레이어 (에이전트 도구)**
- `.claude/skills/` 디렉토리에 생성
- 각 에이전트가 특정 작업 수행을 위한 절차 지식
- Progressive Disclosure 패턴 적용

---

## 3.2 6가지 아키텍처 패턴

### 패턴 선택 플로우차트

```
작업이 순차적으로 의존하는가?
├── YES → 파이프라인
└── NO
    ├── 동일 작업을 병렬 처리하는가?
    │   ├── YES → 팬아웃/팬인
    │   └── NO
    │       ├── 입력에 따라 다른 전문가가 필요한가?
    │       │   ├── YES → 전문가 풀
    │       │   └── NO
    │       │       ├── 생성 후 품질 검증이 반복되는가?
    │       │       │   ├── YES → 생성-검증
    │       │       │   └── NO
    │       │       │       ├── 사전에 흐름 예측이 어려운가?
    │       │       │       │   ├── YES → 감독자
    │       │       │       │   └── NO → 계층적 위임
```

---

### 패턴 1: 파이프라인 (Pipeline)

각 에이전트가 이전 에이전트의 출력을 입력으로 받아 순차 처리.

```
Agent A → Agent B → Agent C → Agent D
  출력₁      출력₂      출력₃      최종
```

**적합한 상황:** 단계별 의존성이 명확한 경우, 중간 결과물 품질 검증 필요한 경우

**IOSYS 적용:** 수학 교육과정 지식 그래프
```
curriculum-parser → standard-extractor → concept-mapper → graph-builder → validator
```

**핵심 설계 원칙:**
- 각 단계 사이에 데이터 형식 계약(schema) 명시
- 중간 단계 실패 시 재시작 지점(checkpoint) 저장
- 병목 단계 식별 후 해당 에이전트에 더 많은 도구 부여

---

### 패턴 2: 팬아웃/팬인 (Fan-out/Fan-in)

오케스트레이터가 동일한 작업을 여러 에이전트에 병렬 분배 후 결과 수집.

```
         orchestrator
        /      |      \
  agent-A  agent-B  agent-C
        \      |      /
         result-merger
```

**적합한 상황:** 독립적인 작업이 대량으로 존재, 처리 속도가 중요, 동일 작업을 다른 관점으로 처리

**IOSYS 적용:** 종합 코드 리뷰
```
review-orchestrator
├── arch-reviewer      (아키텍처)
├── security-reviewer  (보안)
├── perf-reviewer      (성능)
└── style-reviewer     (스타일)
        ↓
  report-merger
```

---

### 패턴 3: 전문가 풀 (Expert Pool)

오케스트레이터가 작업 특성에 따라 적합한 전문 에이전트를 선택적으로 호출.

```
오케스트레이터 (라우터)
├── expert-A  ← 유형 A 요청 시
├── expert-B  ← 유형 B 요청 시
└── expert-C  ← 유형 C 요청 시
```

**적합한 상황:** 입력 유형에 따라 다른 처리 로직 필요, 모든 에이전트를 항상 실행할 필요 없음

**IOSYS 적용:** 음성 채점 시스템 (10개 컴포넌트)
```
speech-router
├── phoneme-scorer      ← 발음 평가
├── fluency-scorer      ← 유창성 평가
├── intonation-scorer   ← 억양 평가
├── content-scorer      ← 내용 평가
└── comprehensive-scorer ← 종합 평가
```

---

### 패턴 4: 생성-검증 (Producer-Reviewer)

생성 에이전트가 결과물을 만들면 검증 에이전트가 품질 검수. 기준 미달 시 재생성.

```
generator → reviewer
    ↑            |
    └── 재생성 ←─┘ (기준 미달 시)
```

**적합한 상황:** 생성 품질이 중요, 환각(hallucination) 위험, 특정 형식 준수 필요

**IOSYS 적용:** AI 문항 은행 (7단계 환각 방지 파이프라인)
```
item-generator → hallucination-checker → qti-formatter → quality-reviewer
      ↑                                                          |
      └──────────────── 기준 미달 시 재생성 ◄──────────────────┘
```

**검증 루프 설계:**
```
최대 재시도 횟수: 3회
실패 기준:
  - 환각 감지율 > 5%
  - QTI 형식 오류 존재
  - 성취기준 미연결

재시도 시: 구체적 피드백 포함해서 재생성 요청
3회 초과 실패: 사람의 검토 요청
```

---

### 패턴 5: 감독자 (Supervisor)

중앙 감독자 에이전트가 전체 흐름을 관리하며 작업을 동적으로 분배.

```
supervisor
├── agent-A  ← 필요 시 호출
├── agent-B  ← 필요 시 호출
└── agent-C  ← 필요 시 호출
```

**적합한 상황:** 사전에 정확한 작업 순서 예측 어려움, 실행 중 결과에 따라 다음 단계 변화

**IOSYS 적용:** QA 자동화 팀
```
qa-supervisor
├── ui-tester          ← 크리티컬 실패 없으면 실행
├── api-tester         ← UI 테스트 통과 후 실행
├── performance-tester ← API 테스트 통과 후 실행
└── qa-reporter        ← 모든 테스트 통과 후 실행
```

---

### 패턴 6: 계층적 위임 (Hierarchical Delegation)

상위 에이전트가 큰 목표를 하위 에이전트에게 작은 목표로 위임. 재귀적 구조.

```
project-manager
├── frontend-lead
│   ├── ui-developer
│   └── test-engineer
├── backend-lead
│   ├── api-developer
│   └── db-engineer
└── devops-lead
```

**적합한 상황:** 매우 큰 작업을 계층적으로 분해, 조직 구조가 계층적인 프로젝트

**위임 시 필수 전달 항목:**
```
1. 목표: 달성해야 할 것
2. 제약: 범위 한계, 하지 말아야 할 것
3. 형식: 출력 결과의 형식과 저장 위치
4. 에스컬레이션: 판단 어려운 상황에서 상위 보고 조건
```

---

## 3.3 실행 모드 결정

```
2개 이상 에이전트 + 협업 필요?
  → YES: 에이전트 팀 모드
           TeamCreate + SendMessage + TaskCreate
           CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 필요

  → NO:  서브 에이전트 모드
           Agent 도구 직접 호출
           단발성 작업, 통신 불필요 시
```

---

## 3.4 파일 디렉토리 설계

### 메타 스킬 구조

```
harness/
├── SKILL.md                              ← 진입점 (500줄 이하)
└── references/
    ├── agent-design-patterns.md          ← Phase 2에서만 로드
    ├── skill-writing-guide.md            ← Phase 4에서만 로드
    ├── orchestrator-template.md          ← Phase 5에서만 로드
    ├── team-examples.md                  ← 필요 시만 로드
    └── qa-agent-guide.md                 ← Phase 6에서만 로드
```

### 생성 산출물 구조

```
프로젝트/
├── .claude/
│   ├── agents/
│   │   ├── orchestrator.md
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── validator.md
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

---

## 3.5 IOSYS 도메인별 패턴 매핑 상세

### 수학 교육과정 지식 그래프 — 파이프라인

```
에이전트 팀:
curriculum-parser → standard-extractor → concept-mapper → graph-builder → validator

스킬 팀:
parse-curriculum-document/   → 교육과정 문서 파싱 방법
extract-achievement-standards/ → 성취기준 추출 형식 및 코드 규칙
map-concept-relations/        → 개념 관계 매핑 기준
validate-graph-integrity/     → 그래프 완결성 검증 기준

핵심 데이터 계약:
{
  "code": "[수학][5-6학년군][수와 연산][01]",
  "content": "성취기준 본문",
  "related_concepts": [...],
  "prerequisite_standards": [...]
}
```

### AI 문항 은행 — 생성-검증

```
에이전트 팀:
item-generator → hallucination-checker → difficulty-calibrator
              → qti-formatter → item-bank-manager

스킬 팀:
generate-assessment-item/  → 문항 생성 지침, 난이도 기준
validate-hallucination/    → 7단계 검증 파이프라인
format-qti3/               → QTI 3.0 XML 형식 규칙
manage-item-bank/          → 중복 확인, 메타데이터 관리
```

### 음성 채점 — 전문가 풀

```
에이전트 팀:
speech-router → {phoneme, fluency, intonation, content, grammar,
                 vocabulary, coherence, task-completion} scorers
             → grade-aggregator

라우팅 로직:
- task_type == "phoneme" → phoneme-scorer 호출
- task_type == "comprehensive" → 모든 scorer 병렬 호출
- task_type 불명확 → 사용자에게 유형 확인 요청
```

> [→ Part 4: 메타 스킬 구현](04-meta-skill-implementation.md)
