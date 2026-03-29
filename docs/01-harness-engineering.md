# Part 1 — Harness Engineering 이론

> [← 인덱스로 돌아가기](../README.md)

---

## 1.1 Harness Engineering이란

2026년 2월 OpenAI가 공개한 내부 엔지니어링 방법론. 5개월 동안 소수의 엔지니어가 Codex 에이전트를 통해 **수동으로 작성된 코드 없이** 100만 줄 규모의 프로덕션 시스템을 구축한 경험에서 탄생했다.

### 핵심 정의

> "에이전트가 신뢰할 수 있게 동작하도록 감싸는 제약(constraints), 피드백 루프(feedback loops), 문서화(documentation), 린터(linters), 생명주기 관리(lifecycle management)의 총체"

**Harness의 어원:** 말(horse)을 제어하는 마구(馬具) — 고삐, 안장, 재갈. 강력하지만 방향을 스스로 찾지 못하는 말(AI 모델)을 올바른 방향으로 안내하는 장치.

```
말(horse)      = AI 모델 — 빠르고 강력하지만 방향을 모름
마구(harness)  = 제약·피드백·문서 — 모델의 힘을 생산적으로 채널링
기수(rider)    = 인간 엔지니어 — 방향을 제시하는 역할
```

---

## 1.2 Context Engineering vs Harness Engineering

커뮤니티에서 가장 많이 논의된 구분.

| 구분 | Context Engineering | Harness Engineering |
|------|---------------------|---------------------|
| 정의 | 에이전트에게 올바른 정보를 주는 것 | 시스템이 무엇을 막고, 측정하고, 수정하는가 |
| 비유 | 신입사원에게 완벽한 온보딩 문서 제공 | CI 파이프라인, 아키텍처 경계, 피드백 루프까지 갖춘 환경 |
| 스코프 | 프롬프트, AGENTS.md, 문서화 | 린터, 구조 테스트, 관측성, 검증 자동화 |
| 실패 패턴 | 한 번 잘못됨 → 컨텍스트 문제 | 반복적 잘못됨, 점진적 악화 → 하네스 문제 |

**자동차 비유:**
```
모델(engine)    = 엔진 — 필수지만 기본으로 제공됨
컨텍스트        = 연료, 오일 교환 — 쉽게 최적화 가능
하네스          = 핸들, 브레이크, 차선 경계, 경고등 — 실제로 차가 굴러가게 하는 것
```

---

## 1.3 Harness의 3대 구성 요소

OpenAI 팀이 실제 구축한 하네스를 Thoughtworks Birgitta가 3가지 범주로 분류.

### 1) 컨텍스트 엔지니어링 (Context Engineering)
- 코드베이스 내 지속 강화되는 지식 베이스
- 관측성 데이터(logs, metrics, traces)에 대한 에이전트 접근
- AGENTS.md를 백과사전이 아닌 목차로 활용
- 진행 상황, 완료된 계획, 기술 부채를 버전 관리

### 2) 아키텍처 제약 (Architectural Constraints)
- LLM 기반 에이전트뿐 아니라 결정론적 린터/구조 테스트로 경계 강제
- 의존성 흐름: `Types → Config → Repo → Service → Runtime → UI`
- 위반은 머지 전 CI에서 차단
- "startup should complete under 800ms" 같은 측정 가능한 제약

### 3) 가비지 컬렉션 (Garbage Collection)
- 문서 불일치나 아키텍처 제약 위반을 주기적으로 찾는 에이전트
- 오래된 문서를 자동으로 발견하고 수정 PR 생성
- 시스템이 수천 번의 변경 후에도 안정성 유지

---

## 1.4 Big Model vs Big Harness 논쟁

커뮤니티의 핵심 긴장 구도.

**Big Model 진영 (OpenAI Noam Brown 등)**
> "reasoning model이 발전하면 복잡한 스캐폴딩이 오히려 악화 요인이 된다. 더 좋은 모델에 같은 질문을 scaffolding 없이 주면 더 잘 된다."

**Big Harness 진영 (Mitchell Hashimoto, LangChain 등)**
> "모델은 이미 상품화되었다. 경쟁 우위는 주변 환경 설계에 있다."

**실증 데이터:**
- LangChain: 동일 모델, 하네스만 변경 → Terminal Bench 2.0에서 52.8% → 66.5% (Top 30 → Top 5)
- Nate B Jones (2026.03): 동일 모델, 하네스만 변경 → 42% → 78%
- revfactory/harness A/B 테스트: 평균 품질 점수 49.5 → 79.3 (+60%), 15/15 승률

**커뮤니티 합의:**
> "모델이 아닌 환경이 병목이다"

---

## 1.5 OpenAI 실험의 핵심 학습

5개월, 엔지니어 3→7명, ~1,500개 PR, 100만 줄 코드.

### 에이전트가 어려움을 겪을 때의 접근법

```
잘못된 접근: "모델이 왜 이렇게 멍청하지?"
올바른 접근: "무슨 도구/제약/문서가 빠져 있는가?"
```

### AGENTS.md 설계 원칙
- 단일 거대 파일 → 목차(table of contents)로 전환
- 모든 상세 내용은 `docs/` 하위 구조화된 파일에
- 린터와 CI가 지식 베이스의 freshness를 강제
- 에이전트 자신이 docs을 업데이트하는 "doc-gardening" 에이전트

### 컨텍스트 관리 핵심 교훈
> "에이전트 관점에서 컨텍스트 내에서 접근할 수 없는 것은 존재하지 않는 것과 같다. Google Docs, Slack 스레드, 사람들의 머릿속에 있는 지식은 시스템에 없다."

---

## 1.6 산업 사례

| 회사 | 구현 내용 | 성과 |
|------|-----------|------|
| **OpenAI** | Codex 에이전트 팀, 수동 코드 없는 개발 | 5개월 100만 줄 코드 |
| **Stripe** | Minions 에이전트, Toolshed MCP 통합 | 주 1,000+ PR 자동화 |
| **Anthropic** | Claude Code 자체 | Claude Code 라인의 90%+가 Claude가 작성 |
| **revfactory** | harness-100, 1,808개 마크다운 파일 | 100개 프로덕션 에이전트 팀 하네스 |

---

## 1.7 IOSYS 프로젝트 적용 관점

| IOSYS 프로젝트 | 적합한 하네스 패턴 | 핵심 제약 |
|----------------|------------------|-----------|
| 수학 교육과정 지식 그래프 | 파이프라인 (4단계 AI) | 성취기준 코드 형식 강제, 그래프 연결성 검증 |
| AI 문항 은행 | 생성-검증 (환각 방지 7단계) | QTI 3.0 스키마, 성취기준 정합성 |
| 음성 채점 시스템 | 전문가 풀 (10개 컴포넌트) | 채점 기준 일관성, 최종 점수 집계 규칙 |
| QTI3 아이템 플레이어 마이그레이션 | 감독자 (150+ 컴포넌트) | Vue3→React18 패턴 일관성 |

---

## 참고 링크

- OpenAI 원문: https://openai.com/index/harness-engineering/
- Martin Fowler 분석: https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html
- Latent.Space 분석: https://www.latent.space/p/ainews-is-harness-engineering-real
- revfactory/harness: https://github.com/revfactory/harness

> [→ Part 2: Claude Code 스킬 이해](02-claude-code-skills.md)
