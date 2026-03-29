# Harness

**Agent Team & Skill Architect** — Claude Code Plugin

도메인에 맞는 전문 에이전트 팀을 설계하고, 에이전트가 사용할 스킬까지 자동 생성하는 메타 스킬.

## 개요

"하네스 구성해줘"라고 말하면, 사용자의 도메인에 맞는 에이전트 정의(`.claude/agents/`)와 스킬(`.claude/skills/`)을 자동 생성한다.

## 설치

### 마켓플레이스를 통한 설치 (권장)

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add your-username/harness

# 2. 플러그인 설치
/plugin install harness@harness
```

### 글로벌 스킬로 직접 설치

```bash
# skills 디렉토리를 ~/.claude/skills/harness/ 에 복사
cp -r skills/harness ~/.claude/skills/harness
```

## 사용법

Claude Code에서 다음과 같이 트리거한다:

```
하네스 구성해줘
이 프로젝트에 맞는 에이전트 팀 설계해줘
하네스 설계해줘
```

## 지원 아키텍처 패턴

| 패턴 | 설명 |
|------|------|
| 파이프라인 | 순차 의존 작업 |
| 팬아웃/팬인 | 병렬 독립 작업 |
| 전문가 풀 | 상황별 선택 호출 |
| 생성-검증 | 생성 후 품질 검수 |
| 감독자 | 중앙 에이전트가 동적 분배 |
| 계층적 위임 | 상위→하위 재귀적 위임 |

## 요구사항

에이전트 팀 기능 활성화 필요:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

## 라이선스

Apache-2.0
