---
name: orchestrator-template
description: "하네스의 오케스트레이터 스킬 템플릿. 생성된 하네스의 전체 워크플로우를 조율."
---

# Orchestrator Skill Template

이 템플릿은 하네스가 생성하는 오케스트레이터 스킬의 표준 형식이다.
오케스트레이터는 모든 에이전트와 스킬을 하나의 워크플로우로 엮는 특수 스킬이다.

## 템플릿 (Agent Teams 모드)

```markdown
---
name: {orchestrator-name}
description: >
  {도메인} {작업}의 전체 워크플로우를 조율. 에이전트 팀을 구성하고 작업을 할당하며
  결과를 통합. "하네스 실행", "워크플로우 시작", "{도메인} 작업" 요청 시,
  "다시 실행", "재실행", "업데이트" 후속 요청 시 사용.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
---

# {Orchestrator Name} — {도메인} 워크플로우 오케스트레이터

{도메인}의 {작업}을 위한 전체 워크플로우를 조율하는 오케스트레이터.

## Phase 0: 컨텍스트 확인

```bash
_WORKSPACE_DIR="{workspace_dir}"
_AGENTS_DIR="{agents_dir}"
_SKILLS_DIR="{skills_dir}"
_HAS_PREVIOUS=false
[ -d "$_WORKSPACE_DIR" ] && _HAS_PREVIOUS=true
echo "HAS_PREVIOUS: $_HAS_PREVIOUS"
mkdir -p "$_WORKSPACE_DIR"
```

- `_workspace/` 존재 + 부분 수정 요청 → **부분 재실행**
- `_workspace/` 존재 + 새 입력 → **새 실행** (기존 _workspace를 `_workspace_prev/`로 이동)
- `_workspace/` 미존재 → **초기 실행**

## Phase 1: {Phase 1 이름}

**실행 모드:** {Agent Teams / Sub-agents / Hybrid}

{Phase 1 설명과 에이전트 호출}

## Phase 2: {Phase 2 이름}

**실행 모드:** {Agent Teams / Sub-agents / Hybrid}

{Phase 2 설명과 에이전트 호출}

## 데이터 전달 프로토콜

| 에이전트 | 입력 | 출력 |
|---------|------|------|
| {에이전트1} | {입력 설명} | {출력 설명} |
| {에이전트2} | {입력 설명} | {출력 설명} |

## 에러 핸들링
- 각 단계 실패 시 1회 재시도
- 재실패 시 해당 결과 없이 진행, 최종 보고서에 누락 명시
- 타임아웃: 단계당 {N}분

## 테스트 시나리오
### 정상 흐름
{정상적인 전체 실행 시나리오}

### 에러 흐름
{에러 발생 시나리오와 대응}
```

## 템플릿 (Sub-agents 모드)

```markdown
---
name: {orchestrator-name}
description: >
  {도메인} {작업}의 워크플로우 조율. 서브 에이전트를 병렬/순차 호출.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
---

# {Orchestrator Name} — {도메인} 워크플로우 오케스트레이터

## Phase 0: 컨텍스트 확인
(Agent Teams 모드와 동일)

## Phase 1: {Phase 1 이름}

**실행 모드:** Sub-agents

{에이전트 병렬 호출 — run_in_background=true}

## Phase 2: {Phase 2 이름}

**실행 모드:** Sub-agents

{순차/병렬 에이전트 호출}
```

## 데이터 전달 프로토콜

기본: 파일 기반 전달 (`_workspace/` 폴더 사용).

파일명 컨벤션: `{phase}_{agent}_{artifact}.{ext}`
예: `01_analyst_findings.md`, `02_reviewer_report.md`

## 에러 핸들링 공통 규칙

| 에러 유형 | 처리 |
|----------|------|
| Agent failure | 1회 재시도, 실패 시 해당 결과 없이 진행 |
| Timeout | phase별 제한 시간 설정, 초과 시 부분 결과 사용 |
| Data conflict | 출처 병기, 삭제 금지, 보고서에 상충 표시 |
| Missing input | 이전 phase 산출물 확인, 없으면 사용자에게 질문 |

## CLAUDE.md 포인터 템플릿

```markdown
## 하네스: {도메인명}

**목표:** {한 줄 목표}
**트리거:** {도메인} 관련 작업 요청 시 `{orchestrator}` 스킬 사용.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
```
