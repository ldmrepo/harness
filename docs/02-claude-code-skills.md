# Part 2 — Claude Code 스킬 이해

> [← 인덱스로 돌아가기](../README.md) | [← Part 1](01-harness-engineering.md)

---

## 2.1 스킬이란 무엇인가

**스킬(Skill) = 절차적 지식의 캡슐화 단위**

Anthropic이 도입한 "조합 가능하고 이식 가능한 온디맨드 절차적 지식 캡슐화 메커니즘". 파일시스템의 디렉토리를 사용해 지시사항과 선택적 리소스(문서, 스크립트, 템플릿)를 능력 모듈로 패키징하며, Claude가 description 매칭을 통해 대화 중 자동으로 호출한다.

### 스킬 vs 다른 구성 요소

| 구분 | 용도 | 예시 |
|------|------|------|
| **스킬(Skill)** | Claude가 어떻게 행동할지 가르침 | 코딩 표준, 분석 워크플로우 |
| **MCP 서버** | Claude에게 새로운 도구 제공 | Slack 메시지 전송, DB 쿼리 |
| **서브에이전트** | 독립적인 컨텍스트에서 작업 실행 | 병렬 분석, 격리된 작업 |
| **프로젝트** | 지속적 배경 컨텍스트 | 팀 지식, 장기 메모리 |

### 스킬이 담아야 하는 것

```
✓ 프로젝트 특유의 규칙과 제약
✓ 도메인 전문 지식 (표준, 형식, 분류체계)
✓ 반복되는 절차 워크플로우
✓ 검증 기준과 피드백 루프
✓ Claude가 갖고 있지 않은 컨텍스트

✗ Claude가 이미 아는 일반 프로그래밍 지식
✗ 라이브러리 기본 사용법
✗ 상식적인 절차 (당연하면 생략)
```

---

## 2.2 Progressive Disclosure 원리

스킬의 핵심 설계 원칙. **컨텍스트 창은 공공재**라는 인식에서 출발.

### 3단계 로딩 구조

```
1단계: 메타데이터 (항상 로드, ~100 토큰)
   └── name + description만 초기 컨텍스트에 존재
   └── Claude가 어떤 스킬을 사용할지 이것만으로 결정

2단계: SKILL.md 본문 (트리거 시 로드)
   └── 스킬이 선택된 후 전체 지시사항 로드
   └── 500줄 이하 유지 권장

3단계: references/ 파일 (온디맨드 로드)
   └── 실제 필요할 때만 bash Read로 로드
   └── 로드 전까지 컨텍스트 토큰 0 소비
```

### 파일 구조 예시

```
bigquery-skill/
├── SKILL.md                    ← 항상 로드 (목차 역할)
└── reference/
    ├── finance.md              ← 매출 질문 시만 로드
    ├── sales.md                ← 영업 질문 시만 로드
    └── product.md              ← 제품 질문 시만 로드
```

**핵심:** SKILL.md에서 매출 관련 질문을 받으면 `reference/finance.md`만 로드. `sales.md`와 `product.md`는 로드되지 않아 컨텍스트 토큰 0 소비.

---

## 2.3 SKILL.md 구조

모든 스킬의 진입점. YAML frontmatter + 마크다운 본문으로 구성.

```yaml
---
name: skill-name           # 소문자, 하이픈만, 64자 이하
description: >             # 반드시 3인칭으로 작성, 1024자 이하
  This skill should be used when {구체적 트리거 조건}.
  {스킬이 하는 것}. Use when {사용 시나리오 키워드}.
---

# 스킬 이름 — 한 줄 요약

## 핵심 정보
{가장 자주 필요한 정보를 첫 번째에}

## 워크플로우
{단계별 절차}

## 참조 자료
- 상세 형식 → `references/format.md` (온디맨드 로드)
- 사용 예시 → `references/examples.md` (온디맨드 로드)
```

---

## 2.4 description 작성 전략

**description이 스킬의 생사를 결정한다.** Claude는 100개 이상의 스킬 중 description만으로 어떤 스킬을 로드할지 결정한다.

### 작성 규칙

1. **반드시 3인칭으로 작성**
   ```yaml
   # 나쁜 예
   description: Helps with curriculum analysis
   
   # 좋은 예
   description: >
     This skill should be used when analyzing Korean curriculum documents,
     extracting achievement standards (성취기준)...
   ```

2. **무엇(what)과 언제(when)를 모두 포함**
   ```yaml
   description: >
     {스킬이 하는 것}. Use when {트리거 키워드, 사용 시나리오}.
   ```

3. **구체적인 트리거 키워드 포함**
   - 사용자가 실제로 쓸 단어를 그대로 포함
   - 한국어/영어 키워드 모두 포함 (다국어 사용 환경)

4. **약간 "pushy"하게 작성**
   - Claude는 기본적으로 스킬을 트리거하지 않으려는 경향
   - "Use when..." 패턴으로 적극 트리거 유도

### 교육 도메인 예시

```yaml
# 성취기준 분석 스킬
name: analyze-curriculum
description: >
  This skill should be used when analyzing Korean K-12 curriculum documents,
  extracting achievement standards (성취기준), identifying content areas,
  or mapping learning progressions from the 2022 revised curriculum
  (2022 개정 교육과정). Use when the user mentions 성취기준 추출,
  교육과정 분석, 학습 위계 분석.

# 문항 생성 스킬
name: generate-assessment-item
description: >
  This skill should be used when generating educational assessment items,
  creating test questions aligned to achievement standards, producing items
  in QTI 3.0 format, or building content for the IOSYS item bank system.
  Use when the user mentions 문항 생성, 문제 출제, QTI, 문항 은행.
```

---

## 2.5 자유도 설정

작업의 성격에 따라 Claude에게 부여하는 자유도를 조정한다.

### 높은 자유도 (텍스트 지시)

여러 접근이 유효하고 맥락에 따라 판단이 달라질 때.

```markdown
## 분석 접근 방식
주어진 문서의 구조와 맥락에 맞게 분석 방향을 결정하라.
키워드, 개념 관계, 위계 구조를 종합적으로 파악한다.
```

### 중간 자유도 (템플릿 + 파라미터)

선호 패턴이 있지만 일부 변형이 허용될 때.

```markdown
## 성취기준 추출 형식
아래 형식을 기본으로 하되, 도메인에 맞게 조정:
```json
{
  "code": "[교과][학년군][내용영역][번호]",
  "content": "성취기준 본문",
  "level": "기본|심화"
}
```
```

### 낮은 자유도 (정확한 스크립트)

형식이 엄격하고 오류 위험이 높을 때.

```markdown
## QTI 3.0 내보내기
반드시 이 스크립트를 실행하라:
```bash
python scripts/export_qti3.py --validate --schema qti3_schema.xsd
```
다른 방법을 사용하지 말 것. 스키마 검증 없이 내보내지 말 것.
```

---

## 2.6 피드백 루프 패턴

출력 품질을 가장 크게 향상시키는 패턴.

```markdown
## 검증 루프

1. {작업 실행}
2. 검증: `python scripts/validate.py output.json`
3. 오류 발생 시:
   - 오류 메시지를 읽고 원인 파악
   - 해당 단계 수정
   - 검증 재실행
4. 검증 통과 전까지 다음 단계로 진행하지 않는다
```

**핵심 설계 원칙:** 린터 오류 메시지는 다음 시도의 컨텍스트가 되도록 작성한다. 시스템이 실수를 막을 뿐 아니라 에이전트를 가르치는 역할도 한다.

---

## 2.7 피해야 할 안티패턴

| 안티패턴 | 문제 | 올바른 방법 |
|----------|------|-------------|
| 중첩 references | 2단계 이상 → 부분 읽기 | SKILL.md에서 직접 1단계만 |
| 500줄 이상 SKILL.md | 컨텍스트 낭비 | references/로 분리 |
| 시간 의존 정보 | 금방 오래됨 | "Old patterns" 섹션으로 분리 |
| 너무 많은 선택지 제시 | Claude 혼란 | 기본값 하나 + 예외 상황 명시 |
| 2인칭 description | 트리거 오류 | 3인칭으로 작성 |
| Windows 경로 (`\`) | Unix 호환 불가 | 항상 `/` 슬래시 사용 |

---

## 2.8 스킬 개발 이터레이션 방법

**Claude A가 스킬을 만들고, Claude B가 테스트한다.**

```
1. Claude A와 함께 작업하며 패턴 파악
   → "이 BigQuery 분석 패턴을 스킬로 만들어줘"

2. Claude A가 SKILL.md 초안 생성
   → 프롬프트 없이 바로 요청 가능 (Claude는 스킬 형식 이해)

3. 테스트 프롬프트 3개 작성
   - 명시적 트리거 (키워드 그대로)
   - 자연스러운 표현 (실제 사용자 언어)
   - 경계 케이스 (트리거 안 되어야 할 것)

4. Claude B (새 세션, 스킬 로드)로 실제 태스크 테스트
   → 관찰: 어디서 실패하는가? 무엇이 빠져 있는가?

5. Claude A에게 피드백 전달 → 스킬 개선
   → 반복
```

---

## 스킬 작성 체크리스트

```
품질 확인:
- [ ] description이 3인칭으로 작성되었는가
- [ ] 트리거 키워드가 구체적으로 포함되었는가
- [ ] SKILL.md 본문이 500줄 이하인가
- [ ] references/ 파일이 모두 존재하는가
- [ ] 중첩 참조가 없는가 (1단계만)
- [ ] 시간 의존적 정보가 포함되지 않았는가
- [ ] 용어가 일관되게 사용되었는가
- [ ] 피드백 루프가 포함되었는가 (품질 중요 작업)
- [ ] 자유도가 적절히 설정되었는가
- [ ] 테스트 프롬프트 3개 이상 검증했는가
```

> [→ Part 3: 하네스 메타 스킬 아키텍처](03-meta-skill-architecture.md)
