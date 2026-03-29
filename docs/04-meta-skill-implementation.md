# Part 4 — 메타 스킬 구현

> [← 인덱스로 돌아가기](../README.md) | [← Part 3](03-meta-skill-architecture.md)

---

## 4.1 전체 파일 구성

```
harness/
├── SKILL.md                              ← 6 Phase 워크플로우 진입점
└── references/
    ├── agent-design-patterns.md          ← 6가지 패턴 상세 + 선택 가이드
    ├── skill-writing-guide.md            ← 에이전트 스킬 작성 원칙
    ├── orchestrator-template.md          ← 오케스트레이터 스킬 템플릿
    ├── team-examples.md                  ← IOSYS 5개 팀 구성 예시
    └── qa-agent-guide.md                 ← 2단계 검증 방법론
```

| 파일 | 줄 수 | 로드 시점 |
|------|-------|---------|
| SKILL.md | 244줄 | 트리거 시 항상 |
| agent-design-patterns.md | 257줄 | Phase 2 (패턴 선택) |
| skill-writing-guide.md | 203줄 | Phase 4 (스킬 생성) |
| orchestrator-template.md | 204줄 | Phase 5 (오케스트레이션) |
| team-examples.md | 198줄 | 참고 요청 시 |
| qa-agent-guide.md | 214줄 | Phase 6 (검증) |

---

## 4.2 SKILL.md — 6 Phase 워크플로우

### Frontmatter

```yaml
---
name: harness
description: >
  This skill should be used when the user wants to "build a harness",
  "design an agent team", "set up a harness", "configure an agent team
  for this project", or needs to generate agent definitions
  (.claude/agents/) and skills (.claude/skills/) tailored to a specific
  domain. Analyzes the project domain, selects an architecture pattern
  from 6 options, and generates a complete, production-ready agent team
  with orchestration, skills, and validation. Use when the user mentions
  harness, agent team, multi-agent system, or asks to automate a complex
  multi-step workflow with specialized agents.
---
```

### 진행 체크리스트

SKILL.md 첫 번째에 체크리스트를 제공해 Claude가 진행 상황을 추적하게 한다.

```markdown
Harness 구성 진행 상황:
- [ ] Phase 1: 도메인 분석 완료
- [ ] Phase 2: 아키텍처 패턴 선택 완료
- [ ] Phase 3: 에이전트 정의 파일 생성 완료
- [ ] Phase 4: 스킬 파일 생성 완료
- [ ] Phase 5: 오케스트레이터 구성 완료
- [ ] Phase 6: 검증 테스트 통과
```

### Phase 1: 도메인 분석

파악 항목 (불명확 시 사용자에게 질문):

```
1. 핵심 목표   — 최종 결과물
2. 주요 작업   — 순차/병렬/조건/반복
3. 필요 전문성 — 각 에이전트 담당 도메인
4. 데이터 흐름 — 에이전트 간 전달 내용
5. 품질 기준   — 성공의 기준
6. 외부 도구   — MCP 서버, API, 파일시스템
```

### Phase 2: 팀 아키텍처 설계

빠른 패턴 선택 기준 (상세는 `references/agent-design-patterns.md`):

```
A → B 순차 의존       → 파이프라인
독립 작업 병렬 처리   → 팬아웃/팬인
입력별 다른 전문가    → 전문가 풀
생성 후 검증 반복     → 생성-검증
동적 흐름 관리 필요   → 감독자
큰 목표 재귀 분해     → 계층적 위임
```

실행 모드 결정:
```
2개 이상 에이전트 + 협업 필요?
  YES → 에이전트 팀 모드 (TeamCreate + SendMessage + TaskCreate)
  NO  → 서브 에이전트 모드 (Agent 도구 직접 호출)
```

### Phase 3: 에이전트 정의 생성

에이전트 파일 템플릿 (`.claude/agents/`에 생성):

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
## 입력
## 출력
## 핵심 작업 흐름
## 품질 기준
## 제약 사항
```

에이전트 수 가이드라인:
- 최소: 2개 (오케스트레이터 + 실행자)
- 권장: 3~5개
- 최대: 7개 (초과 시 오케스트레이션 복잡도 급증)

### Phase 4: 스킬 생성

스킬 작성 상세 가이드는 `references/skill-writing-guide.md` 참조.

스킬 분류:
```
도메인 스킬   → 특정 에이전트 전용
공유 스킬     → 여러 에이전트 공통 사용
오케스트레이터 스킬 → 팀 조율 전용
```

### Phase 5: 통합 및 오케스트레이션

오케스트레이터 템플릿은 `references/orchestrator-template.md` 참조.

오케스트레이터 스킬 생성:
- `.claude/skills/orchestrate-{team-name}/SKILL.md` 생성
- 에이전트 간 데이터 전달 형식 정의
- 에러 핸들링 프로토콜 포함
- 체크포인트 저장 로직 포함

### Phase 6: 검증 및 테스트

검증 상세는 `references/qa-agent-guide.md` 참조.

```markdown
검증 항목:
- [ ] 에이전트 파일 YAML frontmatter 유효성
- [ ] 스킬 description 3인칭 작성 확인
- [ ] SKILL.md 본문 500줄 이하 확인
- [ ] references/ 파일 모두 존재 확인
- [ ] 에이전트 간 데이터 형식 일치 확인
- [ ] 오케스트레이터 스킬 트리거 테스트
- [ ] 에러 케이스 처리 확인
```

---

## 4.3 references/agent-design-patterns.md

**로드 시점:** Phase 2 — 패턴 선택 시

주요 내용:
- 6가지 패턴별 상세 설명 (정의, 적합한 상황, 에이전트 구성 예시)
- 패턴 선택 플로우차트
- 데이터 흐름 다이어그램
- IOSYS 도메인별 적용 예시
- 구현 시 주의사항

---

## 4.4 references/skill-writing-guide.md

**로드 시점:** Phase 4 — 스킬 생성 시

주요 내용:
- 스킬의 본질 (Claude가 이미 아는 것 vs 모르는 것)
- SKILL.md 구조 템플릿
- Progressive Disclosure 적용법 (1단계 참조만 허용)
- description 작성 규칙 (3인칭, 트리거 키워드)
- 자유도 설정 가이드 (높음/중간/낮음)
- 피드백 루프 패턴
- 교육 도메인 스킬 예시 3개
- 스킬 체크리스트

---

## 4.5 references/orchestrator-template.md

**로드 시점:** Phase 5 — 오케스트레이션 구성 시

주요 내용:

### 에이전트 팀 모드 오케스트레이터 템플릿

```markdown
---
name: orchestrate-{team-name}
description: >
  This skill should be used when orchestrating the {팀 이름} agent team...
---

## 팀 구성 표
| 에이전트 | 역할 | 입력 | 출력 |

## 실행 순서 (단계별)

## 에이전트 간 데이터 계약 (JSON 형식)

## 에러 핸들링 프로토콜
- 에이전트 실패 시 원인 분류
- 최대 3회 재시도
- 3회 초과 → 에스컬레이션

## 진행 상황 체크포인트 저장
```

### 에이전트 간 표준 데이터 형식

```json
{
  "status": "success|partial|failed",
  "data": { ... },
  "metadata": {
    "agent": "agent-name",
    "timestamp": "ISO8601",
    "quality_score": 0.0
  },
  "errors": []
}
```

---

## 4.6 references/team-examples.md

**로드 시점:** 참고 요청 시

포함된 예시 팀 5가지:

1. **교육과정 지식 그래프 팀** (파이프라인)
   - 에이전트: curriculum-parser, standard-extractor, concept-mapper, graph-builder, validator
   - 핵심 데이터 형식: 성취기준 JSON 스키마

2. **AI 문항 은행 팀** (생성-검증)
   - 에이전트: item-generator, hallucination-checker, difficulty-calibrator, qti-formatter, item-bank-manager
   - 환각 검증 7단계 기준 포함

3. **음성 채점 시스템 팀** (전문가 풀)
   - 에이전트: speech-router + 10개 scorer + grade-aggregator
   - 라우팅 로직 및 점수 집계 형식

4. **코드 리뷰 팀** (팬아웃/팬인)
   - 에이전트: review-orchestrator, arch/security/perf/style-reviewer, report-merger

5. **QA 자동화 팀** (감독자)
   - 에이전트: qa-supervisor, ui/api/performance/regression-tester, qa-reporter
   - 감독자 의사결정 로직

---

## 4.7 references/qa-agent-guide.md

**로드 시점:** Phase 6 — 검증 시

### Layer 1: 구조 검증

```bash
# 에이전트 파일 YAML 확인
for f in .claude/agents/*.md; do head -10 "$f"; done

# 스킬 줄 수 확인
for f in .claude/skills/*/SKILL.md; do
  lines=$(wc -l < "$f")
  echo "$f: $lines lines"
  [ "$lines" -gt 500 ] && echo "  ⚠️ 500줄 초과!"
done

# 참조 무결성 확인 (참조 파일 실제 존재 여부)
grep -r '\[.*\](references/' .claude/skills/ | ...
```

### Layer 2: 동작 검증

```
테스트 프롬프트 3개 원칙:
1. 명시적 트리거 — description 키워드 그대로
2. 자연스러운 표현 — 실제 사용자 언어
3. 경계 케이스 — 트리거 안 되어야 할 것

드라이런:
1. 오케스트레이터 직접 호출
2. 에이전트 호출 순서 확인
3. 에러 케이스 처리 확인

With/Without 비교:
동일 태스크를 스킬 있을 때/없을 때 비교
```

### 지속적 개선 원칙

```
에이전트 실패 패턴:
- 한 번 잘못됨       → 컨텍스트 문제 → description 개선
- 반복적 잘못됨      → 하네스 문제  → 제약/검증/피드백 루프 추가
- 점진적 악화        → 가비지 컬렉션 문제 → doc freshness 관리
```

---

## 4.8 실제 사용 흐름

```bash
# 1. 설치
cp -r harness/ ~/.claude/skills/harness/

# 2. Claude Code에서 트리거
$ claude

> 수학 교육과정 지식 그래프를 구축하는 하네스를 구성해줘.
  4단계 AI 파이프라인 (Gemini → GPT → Claude Sonnet → Claude Opus)이 필요하고
  181개 성취기준, 843개 성취수준을 처리해야 해.

# 3. Claude가 6 Phase를 실행하며 생성
.claude/agents/curriculum-parser.md    ← 자동 생성
.claude/agents/standard-extractor.md  ← 자동 생성
.claude/agents/concept-mapper.md      ← 자동 생성
.claude/agents/graph-builder.md       ← 자동 생성
.claude/agents/validator.md           ← 자동 생성
.claude/skills/orchestrate-curriculum-graph/SKILL.md  ← 자동 생성
.claude/skills/parse-curriculum/SKILL.md              ← 자동 생성
...
```

> [→ Part 5: 마켓플레이스 등록 및 배포](05-marketplace-deployment.md)
