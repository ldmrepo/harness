# Part 6 — 실전 서비스 구축: 문제은행

> [← 인덱스로 돌아가기](../README.md) | [← Part 5](05-marketplace-deployment.md)

---

## 6.1 전체 흐름 개요

```
Step 1. 환경 준비       ← 설치 확인, 에이전트 팀 기능 활성화
Step 2. 하네스 트리거   ← 구체적인 프롬프트 입력
Step 3. 6 Phase 자동 실행 ← Claude가 에이전트 팀 + 스킬 자동 생성
Step 4. 생성물 검토     ← 데이터 계약·도구 목록 확인 및 커스터마이징
Step 5. 실제 문항 생성  ← 오케스트레이터 스킬 호출
Step 6. 반복 개선       ← 실패 패턴 분석 → 하네스 강화
```

---

## 6.2 Step 1. 환경 준비

### Claude Code 설치 확인

```bash
which claude
# 없으면:
npm install -g @anthropic-ai/claude-code
```

### 에이전트 팀 기능 활성화

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

영구 적용하려면 `.zshrc` 또는 `.bashrc`에 추가:

```bash
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
source ~/.zshrc
```

### 플러그인 설치 확인

Claude Code 실행 후:

```
/plugin list
```

`harness` 가 없으면:

```
/plugin marketplace add ldmrepo/harness
/plugin install harness@harness
```

---

## 6.3 Step 2. 하네스 트리거

Claude Code 터미널에서 `claude` 명령으로 진입 후 아래 프롬프트 입력.
**구체적일수록 더 정확한 에이전트 팀이 생성됩니다.**

```
문제은행 서비스 구축 하네스를 구성해줘.

목표: 2022 개정 교육과정 기반의 AI 문항 자동 생성 및 문항 은행 관리 시스템

요구사항:
- 입력: 성취기준 코드 (예: [수학][5-6학년군][수와 연산][01])
- 출력: QTI 3.0 형식 문항 JSON

처리 단계:
1. 성취기준 분석 → 문항 설계 방향 결정
2. 문항 생성 (선택형/서술형/단답형)
3. 환각 검증 (사실 정확성, 성취기준 정합성, 답안 유일성)
4. QTI 3.0 형식 변환 및 스키마 검증
5. 문항 은행 저장 (중복 확인 포함)

기술 스택: Python/FastAPI 백엔드, PostgreSQL, QTI 3.0 표준

품질 기준:
- 환각 검증 통과율 100%
- QTI 스키마 검증 통과
- 성취기준 정합성 점수 ≥ 0.9
```

---

## 6.4 Step 3. 자동 생성되는 파일 구조

6 Phase가 완료되면 프로젝트 루트에 아래 파일들이 생성됩니다.

### 에이전트 파일 (`.claude/agents/`)

```
.claude/agents/
├── item-bank-orchestrator.md   ← 전체 파이프라인 관리
├── standard-analyzer.md        ← 성취기준 분석 및 문항 설계 방향 결정
├── item-generator.md           ← 문항 초안 생성 (선택형/서술형/단답형)
├── hallucination-checker.md    ← 7단계 환각 검증
├── qti-formatter.md            ← QTI 3.0 XML 변환 및 스키마 검증
└── item-bank-manager.md        ← PostgreSQL 저장, 중복 확인
```

### 스킬 파일 (`.claude/skills/`)

```
.claude/skills/
├── orchestrate-item-bank/
│   └── SKILL.md                     ← 파이프라인 전체 조율
├── analyze-achievement-standard/
│   ├── SKILL.md
│   └── references/
│       └── standard-taxonomy.md     ← 성취기준 분류 체계
├── generate-assessment-item/
│   ├── SKILL.md
│   └── references/
│       ├── item-types.md            ← 문항 유형별 작성 지침
│       └── difficulty-guide.md      ← 난이도 설정 기준
├── validate-hallucination/
│   ├── SKILL.md
│   └── references/
│       └── validation-criteria.md   ← 7단계 검증 기준
├── format-qti3/
│   ├── SKILL.md
│   └── references/
│       └── qti3-schema.md           ← QTI 3.0 XML 형식 규칙
└── manage-item-bank/
    ├── SKILL.md
    └── references/
        └── dedup-rules.md           ← 중복 판단 기준
```

---

## 6.5 Step 4. 생성물 검토 및 커스터마이징

하네스가 생성한 파일은 **초안**입니다. 실제 IOSYS 시스템에 맞게 아래 두 가지를 반드시 검토합니다.

### 검토 1: 에이전트 간 데이터 형식 계약

각 에이전트의 `입력/출력` 섹션이 실제 DB 스키마와 일치하는지 확인합니다.

생성된 파일 예시 (`item-generator.md`):

```yaml
---
name: item-generator
description: >
  This agent should be used when generating educational assessment items
  based on analyzed achievement standards. Produces draft items in
  structured JSON format for validation pipeline.
tools: [Read, Write, Bash]
---

## 입력
```json
{
  "standard_code": "[수학][5-6학년군][수와 연산][01]",
  "standard_content": "덧셈과 뺄셈의 의미를 알고 계산할 수 있다",
  "item_type": "선택형",
  "difficulty": "보통",
  "count": 3
}
```

## 출력
```json
{
  "items": [
    {
      "stem": "문항 본문",
      "options": ["①", "②", "③", "④"],
      "answer": 2,
      "explanation": "해설"
    }
  ]
}
```
```

**수정 포인트:** `options` 필드 형식, `difficulty` 값 범위, `explanation` 포함 여부 등을 실제 시스템 스키마에 맞게 조정합니다.

### 검토 2: tools 목록에 MCP 서버 추가

생성된 에이전트는 기본 도구(`Read`, `Write`, `Bash`)만 포함합니다. 실제 연결이 필요한 MCP 서버를 추가합니다.

```yaml
# item-bank-manager.md 수정 예시
---
name: item-bank-manager
tools: [Read, Write, Bash, mcp__postgres__query, mcp__postgres__execute]
---
```

PostgreSQL MCP가 설정되어 있다면 에이전트가 직접 DB에 쿼리하고 저장할 수 있습니다.

---

## 6.6 Step 5. 실제 문항 생성 실행

### 오케스트레이터 스킬 직접 호출

```
/orchestrate-item-bank [수학][5-6학년군][수와 연산][01]
```

### 또는 자연어로

```
수학 5-6학년군 수와 연산 성취기준 [01]번으로
선택형 문항 3개 생성해서 문항 은행에 저장해줘
```

### 파이프라인 실행 흐름

```
1. standard-analyzer
   입력: "[수학][5-6학년군][수와 연산][01]"
   처리: 성취기준 DB 조회, 핵심 개념 추출, 적합 문항 유형 결정
   출력: standard_analysis.json

2. item-generator
   입력: standard_analysis.json
   처리: 선택형 문항 3개 초안 생성
   출력: draft_items.json

3. hallucination-checker
   입력: draft_items.json
   처리:
     ① 사실 정확성 검증 (교과서 내용 일치 여부)
     ② 성취기준 정합성 점수 계산 (≥ 0.9 기준)
     ③ 답안 유일성 확인
     ④ 선택지 적절성 확인
     ⑤ 언어 수준 적합성 확인
     ⑥ 윤리 검토
     ⑦ 이미지 참조 검증 (있는 경우)
   출력: validated_items.json
   실패 시: item-generator에 구체적 피드백 전달 후 재생성 (최대 3회)

4. qti-formatter
   입력: validated_items.json
   처리: QTI 3.0 XML 변환 → 스키마 검증 (qti3_schema.xsd)
   출력: qti3_items.xml

5. item-bank-manager
   입력: qti3_items.xml
   처리: 중복 문항 확인 → PostgreSQL 저장
   출력: 저장 완료 리포트 (저장된 item_id 목록)
```

### 예상 출력 예시

```json
{
  "pipeline_status": "completed",
  "standard_code": "[수학][5-6학년군][수와 연산][01]",
  "items_generated": 3,
  "items_validated": 3,
  "items_saved": 3,
  "saved_item_ids": ["ITEM_2026_001", "ITEM_2026_002", "ITEM_2026_003"],
  "validation_scores": {
    "hallucination_pass_rate": 1.0,
    "standard_alignment_avg": 0.94,
    "qti_schema_valid": true
  },
  "retry_count": 0
}
```

---

## 6.7 Step 6. 반복 개선 (Harness Engineering 원칙 적용)

### 실패 패턴 분석

```
에이전트가 어려움을 겪을 때 → "무엇이 빠져 있는가?"

한 번 잘못됨      → 컨텍스트 문제  → 해당 스킬 description 또는 지시사항 개선
반복적 잘못됨     → 하네스 문제   → 검증 기준·피드백 루프 강화
점진적 품질 저하  → 문서 부패     → doc-gardening 에이전트 추가
```

### 개선 사례 예시

**문제:** `item-generator`가 성취기준 수준보다 어려운 문항을 반복 생성

**원인 파악:** `generate-assessment-item/references/difficulty-guide.md`에 학년별 어휘 수준 기준이 없음

**해결:**
```bash
# difficulty-guide.md에 학년별 어휘 수준 추가
echo "## 5-6학년군 어휘 수준 기준
- 사용 가능: 교과서 수록 어휘, 일상 생활 용어
- 사용 금지: 중학교 수준 수학 용어, 추상적 개념어" \
  >> .claude/skills/generate-assessment-item/references/difficulty-guide.md
```

**결과:** 스킬 파일만 업데이트하면 에이전트 재배포 없이 즉시 적용

---

## 6.8 다른 서비스로 확장

문제은행과 동일한 방식으로 다른 IOSYS 서비스에도 하네스를 적용할 수 있습니다.

| 서비스 | 트리거 프롬프트 예시 | 권장 패턴 |
|--------|---------------------|-----------|
| 수학 교육과정 지식 그래프 | `"교육과정 지식 그래프 구축 하네스 구성해줘. 4단계 AI 파이프라인 필요..."` | 파이프라인 |
| 음성 채점 시스템 | `"음성 채점 하네스 구성해줘. 10개 채점 컴포넌트가 필요하고..."` | 전문가 풀 |
| QTI3 아이템 플레이어 마이그레이션 | `"Vue3→React18 마이그레이션 하네스 구성해줘. 150개 이상 컴포넌트..."` | 감독자 |
| QA 자동화 | `"IOSYS 플랫폼 QA 자동화 하네스 구성해줘. Playwright 기반..."` | 감독자 |

---

## 핵심 정리

```
1. 하네스는 "한 번 잘 만들면 계속 쓰는 환경"이다
2. 에이전트가 실패하면 → 스킬 파일을 업데이트한다 (코드 수정 아님)
3. 품질 기준은 스킬에 명시적으로 적는다 (측정 가능하게)
4. 진행 상황은 파일에 저장한다 (세션 중단 대비)
5. 점진적으로 개선한다 (처음부터 완벽할 필요 없음)
```

> [← Part 5: 마켓플레이스 등록 및 배포](05-marketplace-deployment.md) | [← 인덱스로 돌아가기](../README.md)
