# 팀 구성 예시

필요 시 로드. 실전 팀 구성 참조용.

---

## 예시 1: 교육과정 지식 그래프 팀 (파이프라인 패턴)

**목표:** 2022 개정 교육과정에서 181개 성취기준, 843개 성취수준을 추출하고 지식 그래프 구축

**에이전트 구성:**
```
.claude/agents/
├── curriculum-parser.md      ← PDF/문서 파싱, 섹션 구조 추출
├── standard-extractor.md     ← 성취기준/성취수준 추출 및 구조화
├── concept-mapper.md         ← 개념 간 관계 매핑, 위계 구조 파악
├── graph-builder.md          ← Neo4j/JSON 형식 지식 그래프 생성
└── curriculum-validator.md   ← 그래프 완결성 검증, 오류 수정
```

**스킬 구성:**
```
.claude/skills/
├── orchestrate-curriculum-graph/
│   └── SKILL.md
├── parse-curriculum-document/
│   ├── SKILL.md
│   └── references/
│       └── document-formats.md   ← HWP, PDF, DOCX 처리 방법
├── extract-achievement-standards/
│   ├── SKILL.md
│   └── references/
│       ├── standard-taxonomy.md  ← 성취기준 분류 체계
│       └── examples.md           ← 추출 예시
└── validate-graph-integrity/
    ├── SKILL.md
    └── references/
        └── quality-criteria.md   ← 그래프 품질 기준
```

**핵심 데이터 형식:**
```json
{
  "achievement_standard": {
    "code": "[수학][5-6학년군][수와 연산][01]",
    "content": "덧셈과 뺄셈의 의미를 알고 계산할 수 있다",
    "subject": "수학",
    "grade_group": "5-6학년군",
    "content_area": "수와 연산",
    "related_concepts": ["덧셈", "뺄셈", "연산"],
    "prerequisite_standards": ["[수학][3-4학년군][수와 연산][01]"]
  }
}
```

---

## 예시 2: AI 문항 은행 팀 (생성-검증 패턴)

**목표:** 성취기준 기반 교육 문항 자동 생성 및 QTI 3.0 형식으로 문항 은행에 저장

**에이전트 구성:**
```
.claude/agents/
├── item-generator.md          ← 성취기준 기반 문항 생성 (멀티모델)
├── hallucination-checker.md   ← 7단계 환각 방지 파이프라인 실행
├── difficulty-calibrator.md   ← 난이도 검증 및 조정
├── qti-formatter.md           ← QTI 3.0 형식 변환 및 검증
└── item-bank-manager.md       ← 문항 은행 저장 및 중복 확인
```

**생성-검증 루프:**
```
item-generator
    ↓ (초안 문항)
hallucination-checker
    ↓ 검증 통과 → difficulty-calibrator
    ↓ 검증 실패 → item-generator (피드백 포함 재생성, 최대 3회)
difficulty-calibrator
    ↓ (난이도 확인된 문항)
qti-formatter
    ↓ (QTI 3.0 XML)
item-bank-manager
    ↓ (저장 완료)
```

**환각 검증 기준 (7단계):**
1. 사실 정확성 — 교과서 내용과 일치하는가
2. 성취기준 정합성 — 해당 성취기준을 측정하는가
3. 이미지 참조 검증 — 이미지가 존재하고 설명과 일치하는가
4. 답안 유일성 — 정답이 하나만 존재하는가
5. 선택지 적절성 — 오답 선택지가 합리적인가
6. 언어 적절성 — 학년 수준에 맞는 언어인가
7. 윤리 검토 — 부적절한 내용이 없는가

---

## 예시 3: 음성 채점 시스템 팀 (전문가 풀 패턴)

**목표:** 학생 음성 녹음을 받아 10개 채점 컴포넌트로 종합 점수 산출

**에이전트 구성:**
```
.claude/agents/
├── speech-router.md           ← 채점 유형 분석 및 에이전트 라우팅
├── phoneme-scorer.md          ← 발음 정확도 평가
├── fluency-scorer.md          ← 유창성 (속도, 멈춤, 리듬) 평가
├── intonation-scorer.md       ← 억양 패턴 평가
├── pronunciation-scorer.md    ← 발음 명확도 평가
├── content-scorer.md          ← 내용 정확성 평가
├── grammar-scorer.md          ← 문법 정확도 평가
├── vocabulary-scorer.md       ← 어휘 적절성 평가
├── coherence-scorer.md        ← 발화 일관성 평가
├── task-completion-scorer.md  ← 과제 완수도 평가
└── grade-aggregator.md        ← 컴포넌트 점수 종합, 최종 성적 산출
```

**라우팅 로직:**
```markdown
채점 유형별 에이전트 조합:
- pronunciation_only: phoneme-scorer, pronunciation-scorer
- fluency_only: fluency-scorer, intonation-scorer
- comprehensive: 모든 scorer 병렬 실행 → grade-aggregator
- custom: {requested_components} 에 명시된 scorer만 실행
```

**점수 집계 형식:**
```json
{
  "student_id": "STU001",
  "task_id": "TASK_001",
  "scores": {
    "phoneme": {"score": 85, "max": 100, "weight": 0.15},
    "fluency": {"score": 72, "max": 100, "weight": 0.20},
    "content": {"score": 90, "max": 100, "weight": 0.25}
  },
  "total_score": 82.5,
  "grade": "B+",
  "feedback": "발음 정확도가 우수하나 유창성 향상 필요"
}
```

---

## 예시 4: 코드 리뷰 팀 (팬아웃/팬인 패턴)

**목표:** 코드베이스를 병렬로 다각도 감사하여 통합 리포트 생성

**에이전트 구성:**
```
.claude/agents/
├── review-orchestrator.md   ← PR 수신, 리뷰어 병렬 호출, 결과 통합
├── arch-reviewer.md         ← 아키텍처 및 설계 패턴 검토
├── security-reviewer.md     ← 보안 취약점 탐지
├── perf-reviewer.md         ← 성능 병목 및 최적화 기회 탐색
├── style-reviewer.md        ← 코드 스타일, 네이밍 컨벤션 검토
└── report-merger.md         ← 4개 리뷰 결과를 우선순위별로 통합
```

**팬아웃 실행:**
```
review-orchestrator가 동시에 4개 에이전트 실행:
  arch-reviewer(PR diff)
  security-reviewer(PR diff)     → 병렬 실행
  perf-reviewer(PR diff)
  style-reviewer(PR diff)
              ↓ (모두 완료 후)
  report-merger → 통합 리포트
```

---

## 예시 5: QA 자동화 팀 (감독자 패턴)

**목표:** IOSYS 플랫폼 릴리즈 전 자동화된 품질 검증 수행

**에이전트 구성:**
```
.claude/agents/
├── qa-supervisor.md         ← 전체 QA 흐름 관리, 동적 테스트 계획
├── ui-tester.md             ← Playwright로 UI 시나리오 테스트
├── api-tester.md            ← REST/WebSocket API 검증
├── performance-tester.md    ← 부하 테스트, 응답시간 측정
├── regression-tester.md     ← 기존 기능 회귀 테스트
└── qa-reporter.md           ← 테스트 결과 리포트 생성
```

**감독자 판단 로직:**
```markdown
## qa-supervisor 의사결정

현재 테스트 결과 분석:
1. 크리티컬 실패 있음? → 즉시 중단, 개발팀 에스컬레이션
2. UI 테스트 통과? → api-tester 실행
3. API 테스트 통과? → performance-tester 실행
4. 성능 기준 미달? → performance-tester 재실행 (최적화 후)
5. 모든 테스트 통과? → qa-reporter로 최종 리포트 생성
```
