# QA 에이전트 가이드

Phase 6에서 로드. 생성된 하네스 검증 및 QA 에이전트 통합 시 참조한다.

---

## 하네스 검증의 두 가지 레이어

```
Layer 1: 구조 검증 (생성 직후)
  → 파일 존재, YAML 유효성, 참조 무결성

Layer 2: 동작 검증 (실행 테스트)
  → 트리거 확인, 드라이런, With/Without 비교
```

---

## Layer 1: 구조 검증 체크리스트

### 에이전트 파일 검증
```bash
# 모든 에이전트 파일 YAML frontmatter 확인
for f in .claude/agents/*.md; do
  echo "=== $f ==="
  head -10 "$f"
done
```

확인 항목:
- [ ] `name` 필드 존재 (소문자, 하이픈만)
- [ ] `description` 필드 존재, 3인칭으로 작성
- [ ] `tools` 필드에 필요한 도구만 포함
- [ ] 각 에이전트 파일에 입력/출력 형식 명시
- [ ] 에이전트 간 데이터 형식이 일치하는지 확인

### 스킬 파일 검증
```bash
# 모든 스킬 SKILL.md 줄 수 확인
for f in .claude/skills/*/SKILL.md; do
  lines=$(wc -l < "$f")
  echo "$f: $lines lines"
  if [ "$lines" -gt 500 ]; then
    echo "  ⚠️  500줄 초과! references/로 분리 필요"
  fi
done
```

확인 항목:
- [ ] 모든 SKILL.md가 500줄 이하
- [ ] `description`이 3인칭으로 작성
- [ ] 트리거 키워드가 구체적으로 포함
- [ ] references/ 파일이 모두 실제로 존재
- [ ] 중첩 참조 없음 (1단계만)

### 참조 무결성 확인
```bash
# SKILL.md에서 참조하는 파일이 실제 존재하는지 확인
grep -r '\[.*\](references/' .claude/skills/ | while IFS=: read file match; do
  ref=$(echo "$match" | grep -oP '(?<=\()references/[^)]+')
  dir=$(dirname "$file")
  if [ ! -f "$dir/$ref" ]; then
    echo "⚠️  참조 파일 없음: $dir/$ref"
  fi
done
```

---

## Layer 2: 동작 검증

### 트리거 테스트

생성된 각 스킬이 예상 프롬프트에서 실제로 트리거되는지 확인한다.

```
테스트 프롬프트 3개 원칙:
1. 명시적 트리거 — description에 있는 키워드를 그대로 사용
2. 자연스러운 표현 — 실제 사용자가 쓸 법한 문장
3. 경계 케이스 — 트리거되지 말아야 할 문장
```

**예시: analyze-curriculum 스킬 트리거 테스트**
```
✓ 트리거 되어야 할 것:
  - "2022 개정 교육과정에서 수학 성취기준을 추출해줘"
  - "이 교육과정 문서의 학습 위계를 분석해줘"
  - "성취기준 분류 체계를 만들어줘"

✗ 트리거 되지 말아야 할 것:
  - "수학 문제 풀어줘" (문항 생성 스킬이 담당)
  - "교육과정 PDF를 다운로드해줘" (파일 작업)
```

### 드라이런 테스트

실제 데이터 없이 흐름만 검증한다:

```
1. 오케스트레이터 스킬 호출
   /orchestrate-{team-name} "테스트 입력"

2. 각 에이전트 호출 확인
   - 올바른 에이전트가 호출되는가
   - 올바른 순서로 실행되는가
   - 에이전트 간 데이터 전달이 정확한가

3. 에러 케이스 확인
   - 빈 입력 처리
   - 잘못된 형식 처리
   - 에이전트 실패 시 재시도
```

### With-Skill vs Without-Skill 비교

같은 태스크를 스킬 있을 때/없을 때 비교해서 개선 효과 측정:

```
Without Skill:
  "2022 개정 수학 교육과정에서 5-6학년군 성취기준을 추출해서 JSON으로 정리해줘"

With Skill (analyze-curriculum):
  "/analyze-curriculum 2022 개정 수학 교육과정 5-6학년군"

비교 기준:
- 출력 형식의 일관성
- 성취기준 코드의 정확성
- 분류 체계의 완결성
- 처리 시간
```

---

## QA 에이전트 통합

하네스 자체에 QA 에이전트를 포함시킬 경우:

### QA 에이전트 파일 템플릿
```markdown
---
name: {team-name}-qa
description: >
  This agent should be used when validating the output quality of
  the {team-name} agent team, running quality checks on generated content,
  or performing acceptance testing before final delivery.
tools: [Read, Bash, Write]
---

# {팀 이름} QA 에이전트

## 검증 책임
- 최종 출력물의 형식 및 완결성 확인
- 비즈니스 규칙 준수 확인
- 엣지 케이스 처리 확인

## 검증 실행
```bash
python scripts/validate_output.py --strict --report qa_report.json
```

## 합격 기준
- 형식 오류: 0건
- 필수 필드 누락: 0건
- 품질 점수: ≥ 0.85

## 실패 시 행동
1. 오류 세부 내용을 qa_report.json에 저장
2. 담당 에이전트에게 구체적 피드백 전달
3. 재작업 후 재검증 요청
```

---

## 지속적 개선 원칙

**"에이전트가 어려움을 겪을 때 → 레포에 무엇이 빠져 있는가를 찾아라"**

```
에이전트 실패 패턴 분석:

한 번 잘못됨 → 컨텍스트 문제 (스킬 description 개선)
반복적 잘못됨 → 하네스 문제 (제약, 검증, 피드백 루프 추가)
점진적 악화 → 가비지 컬렉션 문제 (스킬 freshness 관리)

해결 방법:
1. 실패 원인을 코드/스킬/에이전트 정의에서 찾는다
2. 수정 사항을 레포에 반영한다 (가능하면 에이전트가 직접 수정)
3. 같은 실패가 반복되지 않도록 린터/검증 추가
```

**가비지 컬렉션 에이전트 (선택적):**

장기 운영 하네스에서 문서 부패(drift)를 방지하기 위해 주기적으로 실행:

```markdown
---
name: doc-gardener
description: >
  This agent should be used periodically to scan skill files and agent
  definitions for stale content, outdated references, or drift from
  actual implementation. Use when running scheduled maintenance or when
  skills/agents have become unreliable.
---

## 스캔 대상
1. SKILL.md의 참조 파일이 여전히 존재하는가
2. 에이전트 description의 트리거 키워드가 최신인가
3. 스크립트 경로가 유효한가
4. 데이터 형식 계약이 구현과 일치하는가

## 발견 시 처리
- 경미한 이슈: 자동 수정 PR 생성
- 중요 이슈: 사람에게 에스컬레이션
```
