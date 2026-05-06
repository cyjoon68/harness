# 오케스트레이터 템플릿

> **목적**: 개별 에이전트와 스킬을 워크플로로 묶는 오케스트레이터 스킬을 작성하기 위한 참조 템플릿입니다.
> **적용 대상**: `_harness/skills/` 디렉토리에 위치하는 메타-스킬
> **언어**: 모든 키워드/명령어는 한국어, 코드 식별자는 영어 유지

---

## 오케스트레이터 개요

오케스트레이터는 단일 책임을 가진 개별 에이전트/스킬을 **순차적 또는 병렬 워크플로**로 조율하는 메타-스킬입니다. 각 하위 태스크는 독립적인 에이전트에게 위임되고, 결과는 중앙에서 수집되어 다음 단계로 전달됩니다.

**오케스트레이터가 해결하는 문제:**
- 여러 도메인(디자인, 백엔드, 인프라)이 협력해야 하는 작업
- 단일 에이전트의 컨텍스트 윈도우를 초과하는 복잡한 작업
- 특정 단계의 실패가 전체를 중단시키지 않아야 하는 작업
- 감사(audit) 추적이 필요한 다단계 프로세스

---

## 오케스트레이터 템플릿 (에이전트 팀 모드)

```yaml
---
name: my-orchestrator
description: >
  [2-3문장 설명 — 이 오케스트레이터가 무엇을 하고, 언제 사용하며, 어떤 문제를 해결하는지]
triggers:
  - trigger: "이끌어줘" | "시작해줘" | keywords: ["워크플로", "파이프라인", "오케스트레이션", "..."]
    context: "이 오케스트레이터가 필요한 구체적인 상황 설명"
    priority: high | medium
  - trigger: "특정 명령어"
    context: "다른 진입점 설명"
    priority: medium
---

# [스킬 이름]

## 오케스트레이터 개요

**목표**: [구체적인 목표 한 문장]

**워크플로 다이어그램**:

```
Phase 1: [이름] ──→ Phase 2: [이름] ──→ Phase 3: [이름]
    │                    │                    │
    ├─ Agent A           ├─ Agent C           ├─ Agent E
    ├─ Agent B           ├─ Agent D           └─ Agent F
    └─ (병렬)            └─ (순차)
```

**진입 조건**:
- [조건 1]
- [조건 2]

**산출물**:
- [산출물 1]
- [산출물 2]

---

## Phase 0: 컨텍스트 체크

> 새로 실행인지, 부분 재실행(partial re-run)인지 확인합니다.

```yaml
steps:
  - id: context-check
    description: >
      1. `.harness/state/[task-name]/` 디렉토리 존재 여부 확인
      2. 존재하면 → `_state.json` 읽어서 마지막 완료 Phase 식별
      3. 부분 재실행이면 사용자에게 재개(continue) 여부 확인
      4. 새 실행이면 Phase 1부터 진행
    on_partial_rerun: >
      "이전 실행이 Phase {N}까지 완료되었습니다. 중단한 Phase부터 재개할까요?
      (y = 재개, n = 처음부터, 입력값 = 특정 Phase 번호)"
```

---

## Phase 1: [Phase 이름]

> [Phase 목적 — 한 문장]

### TeamCreate — [팀 이름]

```yaml
agents:
  - name: agent-a
    description: "[담당 역할 — 구체적으로]"
    load_skills: ["skill-a", "skill-b"]
    temperature: 0.3
  - name: agent-b
    description: "[담당 역할]"
    load_skills: ["skill-c"]
    temperature: 0.7
```

### TaskCreate — [태스크 목록]

```yaml
tasks:
  - id: task-1
    agent: agent-a
    description: "[구체적인 태스크 설명]"
    instruction: >
      [에이전트에게 전달할 상세 지시사항.
      마크다운으로 여러 줄 작성 가능.
      참조 파일 경로를 명시할 것.]
    expected_output: "[예상되는 산출물 설명]"
    timeout_ms: 120000
  - id: task-2
    agent: agent-b
    depends_on: [task-1]
    description: "[task-1 결과를 입력으로 사용하는 태스크]"
    instruction: >
      [지시사항. 이전 태스크 결과를 참조하는 방법 명시:
      "task-1의 결과는 {{ task-1.output_path }}에 저장되어 있습니다."]
    timeout_ms: 120000
```

### SendMessage — 동기화/검토 포인트

```yaml
sync_point:
  type: review | notify | checkpoint
  to: user
  message: >
    "[현재까지 진행 상황 요약.
    다음 단계 진행 전 사용자 확인이 필요한 경우 이 시점에 대기]"
```

---

## Phase 2-N: [추가 Phase]

> 위 Phase 1 패턴을 반복합니다. Phase 간 데이터 전달은 아래 **데이터 전달 프로토콜**을 따릅니다.

---

## 데이터 전달 프로토콜

| 전략 | 방식 | 장점 | 단점 | 권장 모드 |
|------|------|------|------|-----------|
| **Message-based** | `SendMessage`로 결과 문자열 전달 | 단순함, 실시간 | 대용량 데이터 부적합 | 서브 에이전트 모드 |
| **Task-based** | `depends_on`으로 이전 태스크 출력 참조 (`{{ task-1.output }}`) | 구조적, 의존성 명확 | 복잡한 DAG에서 난해 | 팀 모드 (primary) |
| **File-based** | JSON/MD 파일로 결과 저장 후 경로 참조 | 대용량, 디버깅 용이 | I/O 오버헤드 | 두 모드 모두 (추천) |
| **Return value-based** | 함수 호출처럼 반환값 전달 | 직관적 | 컨텍스트 제한 | 단순 2-3단계 |

**권장 조합:**

| 모드 | 기본 전략 | 폴백 |
|------|-----------|------|
| 에이전트 팀 모드 | Task-based + File-based | Message-based |
| 서브 에이전트 모드 | File-based + Message-based | Return value-based |

**파일 기반 저장소 규칙:**

```
.harness/state/[task-name]/outputs/
├── phase-1/
│   ├── task-1-result.json
│   └── task-2-result.json
├── phase-2/
│   └── ...
└── _state.json           # 현재까지 완료된 Phase 기록
```

`_state.json` 형식:

```json
{
  "task_name": "my-task",
  "phases_completed": [1],
  "current_phase": 2,
  "started_at": "2026-01-01T00:00:00Z",
  "artifacts": {
    "phase-1": {
      "task-1": ".harness/state/my-task/outputs/phase-1/task-1-result.json"
    }
  }
}
```

---

## 에러 핸들링 전략

| 에러 유형 | 처리 전략 | 구현 |
|-----------|-----------|------|
| **Agent failure** | 1회 재시도 → 재시도 실패 시 skip + 노트 | `max_retry: 1, on_failure: "skip_with_note"` |
| **Timeout** | Phase별 설정 가능한 타임아웃 | `timeout_ms` 파라미터 (기본값 120000) |
| **Conflicting data** | 소스 보존, 삭제 금지 | 충돌 시 `_conflict_*.md` 파일 생성, 원본 유지 |
| **Partial results** | 누락된 부분 문서화 후 진행 | 결과물에 `_missing.md` 첨부, 사용자에게 보고 |

**Agent failure 처리 흐름:**

```
1. Task 실행 → 에러 발생
2. retry_count == 0 → 재시도 (retry_count +1)
3. retry_count == 1 → skip_with_note
4. _state.json의 해당 phase에 "skipped_tasks": ["task-id"] 기록
5. 최종 보고서에 skip된 태스크 목록 포함
```

**Timeout 처리:**

```yaml
timeout_policy:
  default_ms: 120000
  per_task:
    task-heavy-computation: 300000
    task-quick-check: 30000
  on_timeout: "skip_with_note"    # | retry | abort
```

---

## 테스트 시나리오

| 시나리오 | 설명 | 예상 결과 |
|----------|------|-----------|
| 정상 플로우 | 모든 태스크 성공 | 모든 Phase 완료, 최종 산출물 생성 |
| 부분 에이전트 실패 | Agent B 2회 실패 → skip | Phase 완료, skip 기록 포함 |
| 타임아웃 | 특정 태스크 타임아웃 초과 | 해당 태스크 skip, 나머지 정상 진행 |
| 중간 재개 | Phase 2에서 중단 후 재실행 | Phase 2부터 재개, 기존 결과 유지 |
| 전체 재실행 | 사용자가 처음부터 실행 선택 | Phase 1부터, 기존 상태 초기화 |

---

## 오케스트레이터 템플릿 (서브 에이전트 모드)

> `call_omo_agent` + `run_in_background=true`를 사용하는 단순한 대안 템플릿입니다.

```yaml
---
name: my-orchestrator-subagent
description: >
  서브 에이전트 모드를 사용하는 오케스트레이터. 각 단계가 독립적인 에이전트에게 위임됩니다.
triggers:
  - trigger: "서브에이전트 모드로 실행"
    context: "에이전트 팀보다 단순한 워크플로에 적합"
    priority: medium
---

# [스킬 이름] — 서브 에이전트 모드

## Phase 1: [이름]

### Step 1: [태스크 이름]

에이전트에게 다음 태스크를 위임합니다:

`call_omo_agent(description="...", prompt="...", subagent_type="explore", run_in_background=true)`

- **subagent_type**: `explore` (분석/리서치), `librarian` (검색), `oracle` (검증/리뷰)
- **run_in_background**: 반드시 `true`
- **timeout**: `background_output(task_id=..., timeout=...)`로 결과 수신

### Step 2: 결과 수집

```yaml
await:
  - task_id_A (from step 1)
  - task_id_B (from step 1, if parallel)

process:
  description: "수집된 결과를 다음 Phase 입력용으로 가공"
  output_path: ".harness/state/[task-name]/outputs/phase-1/"
```

### Step 3: 진행 보고

```yaml
sync_point:
  type: notify
  to: user
  message: "Phase 1 완료. 결과 요약: ..."
```

---

## Phase 2-N: [이름]

> Phase 1 패턴을 반복합니다. 각 Phase는 이전 Phase의 `output_path`에서 데이터를 읽습니다.

---

## 하이브리드 패턴 템플릿

> 팀 모드와 서브 에이전트 모드를 Phase별로 혼합합니다.

```yaml
phase_1:
  mode: team                    # 팀 모드 — 병렬 탐색/분석
  agents:
    - name: researcher          # explore 계열
    - name: analyzer            # oracle 계열

sync_point:
  type: checkpoint              # 결과 취합 후 사용자 검토

phase_2:
  mode: subagent                # 서브 에이전트 모드 — 단일 집중 태스크
  subagent_type: oracle
  description: "Phase 1 결과 검증"

phase_3:
  mode: team                    # 팀 모드 — 구현
  agents:
    - name: implementor
    - name: reviewer
```

**하이브리드 사용 기준:**

| 조건 | 권장 모드 |
|------|-----------|
| 병렬 탐색/분석 필요 | 팀 모드 |
| 단일 집중 검증/리뷰 | 서브 에이전트 모드 |
| 다수 파일 생성 구현 | 팀 모드 |
| 사용자와의 긴 대화 필요 | 서브 에이전트 모드 |
| 단계 간 의존성 복잡 | 팀 모드 (Task-based) |

---

## 팀 크기 가이드라인

| 규모 | 에이전트 수 | Phase당 태스크 수 | 적합한 작업 |
|------|-------------|-------------------|-------------|
| **Small** | 2-3 | 2-5 | 단순 분석, 코드 리뷰, 문서 생성 |
| **Medium** | 3-5 | 3-8 | 기능 구현 (프론트+백엔드+테스트) |
| **Large** | 5-7 | 5-12 | 전체 마이그레이션, 대규모 리팩터링 |

**규모별 원칙:**
- Small: 모든 에이전트가 동일 Phase에서 병렬 실행 가능
- Medium: Phase 분할 필요 (2-3 Phase), 일부 순차 의존성
- Large: 3+ Phase, 복잡한 DAG, 체크포인트 필수

**오버헤드 경고:** 7명 이상의 에이전트는 관리 오버헤드가 급증합니다. 7명 이상이 필요한 경우, 계층적 오케스트레이터(하위 오케스트레이터에 위임)를 고려하세요.

---

## CLAUDE.md 하네스 포인터 등록

> 프로젝트 `CLAUDE.md` (또는 `AGENTS.md`)에 추가할 하네스 참조 템플릿입니다.

```markdown
## Harness Meta-Skills

이 프로젝트는 `_harness/` 디렉토리 아래 메타-스킬(오케스트레이터)을 관리합니다.

### 사용 가능한 오케스트레이터

| 스킬 | 설명 | 사용법 |
|------|------|--------|
| `_harness/skills/my-orchestrator/SKILL.md` | [간략 설명] | `이끌어줘 [작업명]` |

### 상태 저장소

오케스트레이터 실행 상태는 `.harness/state/`에 저장됩니다.
- 부분 재실행(partial re-run)이 가능합니다.
- 상태 파일을 수동으로 삭제하면 전체 재실행됩니다.

### 오케스트레이터 추가

새 오케스트레이터를 추가하려면:
1. `_harness/skills/[name]/SKILL.md` 생성
2. 위 템플릿 참조하여 작성
3. 이 테이블에 등록
```
