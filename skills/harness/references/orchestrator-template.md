# 오케스트레이터 템플릿

Phase 5에서 로드. 에이전트 팀 조율 스킬 작성 시 참조한다.

---

## 오케스트레이터의 역할

오케스트레이터는 **지휘자**다. 직접 작업을 수행하지 않고, 적절한 에이전트에게 올바른 순서로 올바른 정보를 전달한다.

**오케스트레이터가 하는 것:**
- 전체 목표를 하위 작업으로 분해
- 각 에이전트에게 작업 할당 및 컨텍스트 전달
- 에이전트 결과물 수집 및 품질 확인
- 실패 감지 및 재시도 결정
- 최종 결과물 통합 및 사용자에게 전달

**오케스트레이터가 하지 않는 것:**
- 직접 도메인 작업 수행 (전문 에이전트에게 위임)
- 세부 구현 결정 (각 에이전트의 책임)

---

## 에이전트 팀 모드 오케스트레이터 스킬 템플릿

```markdown
---
name: orchestrate-{team-name}
description: >
  This skill should be used when orchestrating the {팀 이름} agent team,
  coordinating {주요 작업}, or managing the end-to-end workflow of {목표}.
  Use when the user asks to run {팀 이름} pipeline or execute {도메인} workflow.
---

# {팀 이름} 오케스트레이터

## 팀 구성

| 에이전트 | 역할 | 입력 | 출력 |
|---------|------|------|------|
| {agent-1} | {역할} | {입력 형식} | {출력 형식} |
| {agent-2} | {역할} | {입력 형식} | {출력 형식} |
| {agent-3} | {역할} | {입력 형식} | {출력 형식} |

## 실행 순서

```
Step 1: {agent-1} 실행
  입력: $ARGUMENTS
  출력 저장: /tmp/{team-name}/step1_output.json
  완료 기준: {검증 조건}

Step 2: {agent-2} 실행
  입력: /tmp/{team-name}/step1_output.json
  출력 저장: /tmp/{team-name}/step2_output.json
  완료 기준: {검증 조건}

...

최종: 결과물 통합 및 사용자에게 전달
```

## 에이전트 간 데이터 계약

### Step 1 → Step 2 전달 형식
```json
{
  "status": "success|partial|failed",
  "data": { ... },
  "metadata": {
    "agent": "{agent-1}",
    "timestamp": "ISO8601",
    "quality_score": 0.0~1.0
  },
  "errors": []
}
```

## 에러 핸들링 프로토콜

```
에이전트 실패 시:
1. 실패 원인 분류
   - 입력 데이터 오류 → 이전 단계 재실행
   - 처리 오류 → 현재 에이전트 재시도 (최대 3회)
   - 시스템 오류 → 사람에게 에스컬레이션

2. 재시도 시 피드백 전달
   - 어떤 부분이 실패했는지
   - 수정 방향 제안

3. 3회 재시도 후에도 실패 → 사용자에게 보고 후 중단
```

## 진행 상황 저장

```bash
# 체크포인트 저장 (세션 중단 시 재개 가능)
echo '{"phase": "step2", "completed": ["step1"], "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"}' \
  > /tmp/{team-name}/checkpoint.json
```
```

---

## 서브에이전트 모드 오케스트레이터 스킬 템플릿

단발성 작업이나 에이전트 간 통신이 필요 없는 경우 사용.

```markdown
---
name: run-{task-name}
description: >
  This skill should be used when executing {단일 작업} using a subagent,
  running {작업 유형} in an isolated context, or performing {목적} without
  requiring inter-agent communication.
context: fork
agent: Explore  # 또는 Edit, General
---

# {작업 이름}

## 작업 목표
$ARGUMENTS 에서 전달된 입력을 처리해 {목표}를 달성한다.

## 실행 단계
1. {step 1}
2. {step 2}
3. 결과를 {형식}으로 저장

## 완료 기준
- {기준 1}
- {기준 2}
```

---

## 실제 오케스트레이터 예시: 교육과정 지식 그래프 팀

```markdown
---
name: orchestrate-curriculum-graph
description: >
  This skill should be used when building a knowledge graph from Korean curriculum
  documents, orchestrating the curriculum-graph agent team, or running the
  end-to-end pipeline from curriculum analysis to graph generation.
  Use when the user mentions 교육과정 지식 그래프 생성, 성취기준 그래프, 지식 그래프 파이프라인.
---

# 교육과정 지식 그래프 오케스트레이터

## 팀 구성

| 에이전트 | 역할 | 담당 모델 |
|---------|------|---------|
| curriculum-parser | 교육과정 문서 파싱 | Claude Sonnet |
| standard-extractor | 성취기준 추출 | Gemini 2.5 Pro |
| concept-mapper | 개념 관계 매핑 | GPT-5 |
| graph-builder | 지식 그래프 생성 | Claude Sonnet |
| validator | 검증 및 오류 수정 | Claude Opus |

## 파이프라인 실행 순서

```
1. curriculum-parser 실행
   입력: 교육과정 PDF/문서
   출력: /tmp/curriculum-graph/parsed_sections.json

2. standard-extractor 실행
   입력: parsed_sections.json
   출력: /tmp/curriculum-graph/achievement_standards.json
   검증: 총 성취기준 수 ≥ 예상값의 90%

3. concept-mapper 실행
   입력: achievement_standards.json
   출력: /tmp/curriculum-graph/concept_relations.json
   검증: 모든 성취기준이 최소 1개 개념과 연결

4. graph-builder 실행
   입력: concept_relations.json
   출력: /tmp/curriculum-graph/knowledge_graph.json
   검증: 그래프 연결성 확인, 고립 노드 없음

5. validator 실행
   입력: knowledge_graph.json
   출력: /tmp/curriculum-graph/final_graph.json
   검증: 전체 품질 점수 ≥ 0.85
```

## 에러 핸들링

단계별 실패 시 해당 단계만 재시작. 3회 실패 시 사용자에게 보고.
중간 결과는 /tmp/curriculum-graph/ 에 저장해 재시작 가능.
```

---

## 오케스트레이터 설계 원칙

1. **상태를 파일에 저장한다** — 메모리에만 의존하면 세션 중단 시 모든 진행이 사라진다
2. **각 단계의 완료 기준을 명시한다** — "잘 됐나?"가 아니라 측정 가능한 기준
3. **실패를 정상으로 취급한다** — 재시도 로직은 옵션이 아닌 필수
4. **피드백을 구조화한다** — 에러 메시지가 다음 시도의 컨텍스트가 된다
5. **컨텍스트 창을 보호한다** — 중간 결과는 파일로 저장, 요약본만 컨텍스트에 유지
