# 스킬 작성 가이드

Phase 4에서 로드. 에이전트용 스킬 작성 시 참조한다.

---

## 스킬의 본질

**스킬 = 절차적 지식의 캡슐화 단위**

스킬은 Claude가 이미 알고 있는 것을 반복하지 않는다. Claude가 갖고 있지 않은 컨텍스트만 추가한다.

```
스킬이 담아야 하는 것:
✓ 프로젝트 특유의 규칙과 제약
✓ 도메인 전문 지식 (표준, 형식, 분류체계)
✓ 반복되는 절차 워크플로우
✓ 검증 기준과 피드백 루프

스킬이 담지 말아야 하는 것:
✗ Claude가 이미 아는 일반 프로그래밍 지식
✗ 라이브러리 기본 사용법 (공식 문서로 대체)
✗ 상식적인 절차 (단계가 당연하면 생략)
```

---

## SKILL.md 구조

```yaml
---
name: {skill-name}           # 소문자, 하이픈만, 64자 이하
description: >               # 반드시 3인칭으로 작성
  This skill should be used when {구체적 트리거 조건}.
  {스킬이 하는 것}. Use when {사용 시나리오 키워드}.
---

# {스킬 이름} — {한 줄 요약}

## 핵심 정보 (항상 여기서 시작)
{가장 자주 필요한 정보를 첫 번째에}

## 워크플로우
{단계별 절차}

## 참조 자료
- 상세 형식 → [references/format.md](references/format.md)
- 사용 예시 → [references/examples.md](references/examples.md)
```

---

## Progressive Disclosure 적용법

**핵심 원칙: SKILL.md는 목차, references/는 내용**

```
SKILL.md (500줄 이하, 항상 로드)
└── references/
    ├── domain-spec.md     ← 도메인 특화 규칙 (해당 작업 시만 로드)
    ├── examples.md        ← 사용 예시 (예시 필요 시만 로드)
    └── error-guide.md     ← 오류 처리 (오류 발생 시만 로드)
```

**잘못된 구조 (중첩 참조):**
```
SKILL.md → references/main.md → references/detail.md → references/sub.md
```
2단계 이상 중첩은 Claude가 부분 읽기(head -100)를 사용해 정보 누락이 발생한다.
**references/ 는 SKILL.md에서 직접 참조, 1단계만 허용.**

---

## description 작성 규칙

```yaml
# 나쁜 예시
description: Helps with curriculum analysis

# 좋은 예시
description: >
  This skill should be used when analyzing Korean curriculum documents,
  extracting achievement standards (성취기준), mapping concept relationships,
  or building knowledge graphs from the 2022 revised curriculum. Use when
  the user mentions 교육과정, 성취기준, 교육과정 분석, or knowledge graph generation.
```

**트리거 키워드를 명시적으로 포함한다.** Claude는 100개 이상의 스킬 중 description만으로 어떤 스킬을 로드할지 결정한다.

---

## 자유도 설정

**높은 자유도 (텍스트 지시):**
```markdown
## 분석 접근 방식
주어진 문서의 구조와 맥락에 맞게 분석 방향을 결정하라.
키워드, 개념 관계, 위계 구조를 종합적으로 파악한다.
```
→ 여러 접근이 유효하고 맥락에 따라 판단이 달라질 때 사용

**중간 자유도 (템플릿 + 파라미터):**
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
→ 선호 패턴이 있지만 일부 변형이 허용될 때 사용

**낮은 자유도 (정확한 스크립트):**
```markdown
## QTI 3.0 내보내기
반드시 이 스크립트를 실행하라:
```bash
python scripts/export_qti3.py --validate --schema qti3_schema.xsd
```
다른 방법을 사용하지 말 것. 스키마 검증 없이 내보내지 말 것.
```
→ 형식이 엄격하고 오류 위험이 높을 때 사용

---

## 피드백 루프 패턴

```markdown
## 검증 루프

1. {작업 실행}
2. 검증: `python scripts/validate.py output.json`
3. 오류 발생 시:
   - 오류 메시지를 읽고 원인 파악
   - 해당 단계 수정
   - 검증 재실행
4. **검증 통과 전까지 다음 단계로 진행하지 않는다**
```

이 패턴이 출력 품질을 가장 크게 향상시킨다.

---

## 교육 도메인 스킬 예시

### 성취기준 분석 스킬 (analyze-curriculum)
```yaml
---
name: analyze-curriculum
description: >
  This skill should be used when analyzing Korean K-12 curriculum documents,
  extracting achievement standards (성취기준), identifying content areas,
  or mapping learning progressions from the 2022 revised curriculum (2022 개정 교육과정).
  Use when the user mentions 성취기준 추출, 교육과정 분석, 학습 위계 분석.
---
```

### 문항 생성 스킬 (generate-assessment-item)
```yaml
---
name: generate-assessment-item
description: >
  This skill should be used when generating educational assessment items,
  creating test questions aligned to achievement standards, producing items
  in QTI 3.0 format, or building content for the IOSYS item bank system.
  Use when the user mentions 문항 생성, 문제 출제, QTI, 문항 은행.
---
```

### 환각 검증 스킬 (validate-hallucination)
```yaml
---
name: validate-hallucination
description: >
  This skill should be used when checking AI-generated educational content
  for hallucinations, verifying factual accuracy of assessment items,
  validating alignment between items and achievement standards, or running
  the 7-stage hallucination prevention pipeline. Use when the user mentions
  환각 검증, 사실 확인, 문항 검증, 성취기준 정합성.
---
```

---

## 스킬 체크리스트

생성한 스킬을 배포하기 전 확인:

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
```
