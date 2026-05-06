---
name: harness
description: >
  하네스를 구성합니다. 프로젝트 도메인에 맞는 전문 에이전트 팀을 설계하고, 각 에이전트가 사용할 스킬을
  생성하는 메타 스킬. (1) "하네스 구성해줘", "하네스 구축해줘" 요청 시, (2) "에이전트 팀 만들어줘",
  "전문가 팀 구성해줘" 요청 시, (3) 새 도메인/프로젝트의 자동화 체계 구축 시,
  (4) 기존 하네스 확장/재구성 시, (5) "하네스 점검", "하네스 감사" 등 유지보수 요청 시 사용.
  (6) "딥 모드", "deep mode", "인터뷰", "질문", "명세" 키워드와 함께 사용하면 심층 도메인 분석 후 하네스 생성.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
  - Task
triggers:
  - 하네스 구성
  - 하네스 구축
  - 에이전트 팀
  - 팀 구성
  - 전문가 팀
  - 딥 모드
  - deep mode
  - 하네스 점검
  - 하네스 감사
---

# Harness — OpenCode Agent Team & Skill Factory

프로젝트 도메인에 맞는 전문 에이전트 팀을 설계하고, 각 에이전트가 사용할 스킬을 생성하는 메타 스킬.
gstack 스타일의 글로벌 스킬로 동작하며, 생성된 하네스는 프로젝트 로컬(`.agents/skills/`)에 구성된다.

**핵심 원칙:**
1. **전역 스킬 → 로컬 하네스**: 이 스킬(전역)이 하네스(프로젝트 로컬)를 생성한다.
2. **Deep Mode 우선**: Ouroboros 방법론으로 숨겨진 요구사항을 발굴한 후 생성한다.
3. **진화하는 시스템**: 하네스는 고정물이 아니라 사용 피드백으로 계속 진화한다.
4. **모듈성**: 에이전트(누가)와 스킬(어떻게)을 분리하여 재사용성을 높인다.

---

## Preamble

```bash
_HARNESS_DIR="$([ -d "$(pwd)/.agents/skills/harness" ] && echo "$(pwd)/.agents/skills/harness" || echo "")"
_PROJECT_DIR="$(pwd)"
_AGENTS_DIR="$_PROJECT_DIR/.agents"
_SKILLS_DIR="$_PROJECT_DIR/.agents/skills"
echo "PROJECT_DIR: $_PROJECT_DIR"
echo "AGENTS_DIR: $_AGENTS_DIR"
echo "SKILLS_DIR: $_SKILLS_DIR"

# 기존 하네스 확인
_EXISTING_HARNESS=false
if [ -d "$_AGENTS_DIR" ] && (ls "$_AGENTS_DIR"/*.md 2>/dev/null | head -1 | grep -q .); then
  _EXISTING_HARNESS=true
fi
echo "EXISTING_HARNESS: $_EXISTING_HARNESS"

# Deep mode 확인
_DEEP_MODE=false
case " $* " in
  *"딥"*|*"deep"*|*"인터뷰"*|*"interview"*|*"명세"*|*"spec"*) _DEEP_MODE=true ;;
esac
# 사용자 입력에 deep mode 키워드가 있는지도 확인
echo "DEEP_MODE: $_DEEP_MODE"

# 초기화
mkdir -p "$_SKILLS_DIR"
```

---

## 워크플로우

### Phase 0: 현황 감사 & 실행 모드 결정

#### 0-1. 기존 하네스 감사
1. `$_AGENTS_DIR/` — 에이전트 정의 파일 목록 확인
2. `$_SKILLS_DIR/` — 스킬 파일 목록 확인
3. 현황에 따라 분기:
   - **신규 구축**: 에이전트/스킬 없음 → Phase 1부터 전체 실행
   - **기존 확장**: 기존 하네스 존재 + 추가 요청 → 영향받는 Phase만 실행
   - **운영/유지보수**: 감사/수정/동기화 요청 → Phase 7로 이동

#### 0-2. 실행 모드 선택

| 모드 | 설명 | 트리거 |
|------|------|--------|
| **Standard** (기본) | 도메인 분석 → 팀 설계 → 생성 → 검증 | 일반 요청 |
| **Deep Mode** | Ouroboros 인터뷰 → 모호성 점수 → 설계 → 생성 | "딥 모드", "인터뷰" 포함 요청 |
| **Quick** | 인터뷰 생략, 직접 질문으로 바로 생성 | "빠르게", "간단히" 포함 요청 |

사용자 요청에 `_DEEP_MODE=true`이면 Deep Mode로 진행.
그 외는 Standard 모드로 진행하되, 도메인이 복잡하거나 불명확하면 Deep Mode로 전환을 제안한다.

---

### Phase 0-D: Deep Mode — Ouroboros Domain Interview (Deep Mode 전용)

이 Phase는 Deep Mode에서만 실행된다. Ouroboros 방법론으로 도메인을 심층 분석한다.

#### Step D-1: 소크라테스식 인터뷰

다음 질문들을 순차적으로 던져서 사용자의 숨겨진 가정을 드러낸다. 한 번에 모든 질문을 하지 말고, 답변에 따라 다음 질문을 선택한다.

**핵심 질문 세트** (필수):
1. "정확히 무엇을 만들려고 하나요? 한 문장으로 설명해주세요."
2. "이것이 해결하는 진짜 문제는 무엇인가요? 증상이 아니라 근본 원인을 말씀해주세요."
3. "이미 이 영역에서 시도해본 것이 있나요? 무엇이 잘 안됐나요?"
4. "성공했다는 것을 어떻게 알 수 있나요? 측정 가능한 기준이 있나요?"
5. "이 프로젝트의 제약 조건은 무엇인가요? (시간, 예산, 기술, 인력)"
6. "반대 상황이 진짜라면요? 가장 확신하는 가정에 의문을 제기해보겠습니다."
7. "이 도메인에서 가장 복잡한 부분은 무엇이라고 생각하나요?"

**인터뷰 원칙:**
- 질문만 한다. 절대 해결책을 제시하지 않는다.
- 모호한 답변에는 follow-up 질문으로 파고든다.
- 사용자의 용어를 그대로 사용하여 질문한다.
- 5-7개의 질문 후에는 Seed 단계로 진행할 준비가 되었는지 확인한다.

#### Step D-2: 모호성 점수 산정

답변을 바탕으로 모호성 점수를 계산한다:

```
Ambiguity = 1 − Σ(clarityᵢ × weightᵢ)
```

| 차원 | 질문 | Greenfield 가중치 |
|------|------|------------------|
| **목표 명확도** | "목표가 구체적인가, 모호한가?" (0.0~1.0) | 40% |
| **제약 명확도** | "제한 사항이 명확히 정의되었는가?" (0.0~1.0) | 30% |
| **성공 기준** | "결과가 측정 가능한가?" (0.0~1.0) | 30% |

점수 산정 방식:
- 각 차원을 LLM이 0.0~1.0로 평가한다 (temperature 0.1)
- 가중치를 곱하여 합산한다
- `Ambiguity ≤ 0.2`면 Seed 생성 가능
- `Ambiguity > 0.2`면 추가 질문으로 더 파고든다

**임계값 게이트:**
- Ambiguity ≤ 0.20 → ✅ Seed 생성 진행
- Ambiguity 0.21~0.40 → ⚠️ 추가 질문 2-3개 후 재측정
- Ambiguity > 0.40 → 🔄 추가 질문 5개 후 재측정, 3회 실패 시 사용자에게 "더 구체화가 필요합니다" 안내

#### Step D-3: Seed 생성

인터뷰 결과를 확정된 명세(Seed)로 정리한다:

```markdown
# {프로젝트명} — Seed Specification

## 목표
{한 문장 목표}

## 범위
- In scope: {명확히 포함}
- Out of scope: {명확히 제외}

## 주요 작업 유형
1. {작업 유형 1} — 생성/분석/검증/편집
2. {작업 유형 2} — 생성/분석/검증/편집

## 제약 조건
- {제약 1}
- {제약 2}

## 성공 기준
- [ ] {측정 가능한 기준 1}
- [ ] {측정 가능한 기준 2}

## 모호성 점수
- Goal clarity: {점수}
- Constraint clarity: {점수}
- Success criteria: {점수}
- **Ambiguity: {최종 점수}** (≤ 0.2 → ✅)
```

사용자에게 Seed를 보여주고 확인을 받는다. 수정 요청 시 반영 후 재확인.

---

### Phase 1: 도메인 분석 (Standard Mode)

Deep Mode를 거치지 않은 Standard Mode에서 도메인을 분석한다.

1. 사용자 요청에서 도메인/프로젝트 파악
2. 핵심 작업 유형 식별 (생성, 검증, 편집, 분석 등)
3. 프로젝트 코드베이스 탐색 — 기술 스택, 주요 모듈 파악
4. 필요한 에이전트 수와 역할 추정

**산출물:** 도메인 분석 요약 (Phase 2의 입력)

---

### Phase 2: 팀 아키텍처 설계

#### 2-1. 아키텍처 패턴 선택

작업 특성에 따라 6가지 패턴 중 선택:

| 패턴 | 설명 | 적합한 작업 |
|------|------|-----------|
| **파이프라인** | 순차 의존 작업 | 문서 생성, 데이터 처리 |
| **팬아웃/팬인** | 병렬 독립 작업 → 통합 | 리서치, 코드 리뷰 |
| **전문가 풀** | 상황별 선택 호출 | 문의 처리, 분류 |
| **생성-검증** | 생성 후 품질 검수 | 콘텐츠 제작, 번역 |
| **감독자** | 중앙 에이전트가 동적 분배 | 대규모 마이그레이션 |
| **계층적 위임** | 상위→하위 재귀적 위임 | 풀스택 개발 |

상세: `references/agent-design-patterns.md` 참조.

#### 2-2. 실행 모드 선택

| 모드 | 언제 사용 |
|------|----------|
| **Agent Teams** (기본) | 2+ 에이전트 협업, 실시간 조율 필요 |
| **Sub-agents** (대안) | 단일 에이전트, 결과만 반환 |
| **하이브리드** | Phase별 특성이 다를 때 |

상세 비교: `references/agent-design-patterns.md` 참조.

#### 2-3. 에이전트 목록 확정

패턴과 모드 결정 후 에이전트 목록 작성:
- 각 에이전트의 역할과 책임
- 에이전트 간 데이터 흐름
- 필요한 스킬 목록

---

### Phase 3: 에이전트 정의 생성

각 에이전트를 `$_AGENTS_DIR/{name}.md` 파일로 정의한다.

**필수 섹션:**
- 핵심 역할
- 작업 원칙 (3-5개)
- 입력/출력 프로토콜
- 에러 핸들링
- 협업 규칙 (팀 모드인 경우 팀 통신 프로토콜 포함)

**템플릿:**

```markdown
---
name: {agent-name}
description: "{역할 설명}"
---

# {Agent Name} — {역할 한 줄}

당신은 {도메인}의 {역할} 전문가입니다.

## 핵심 역할
1. {역할1}
2. {역할2}

## 작업 원칙
- {원칙1}
- {원칙2}

## 입력/출력 프로토콜
- 입력: {입력 설명}
- 출력: {출력 설명}

## 에러 핸들링
- {실패 시 행동}

## 협업
- {다른 에이전트와 관계}
```

에이전트 정의 템플릿과 전체 예시는 `references/harness-templates/agent-definition.md` 참조.

---

### Phase 4: 스킬 생성

각 에이전트가 사용할 스킬을 `$_SKILLS_DIR/{name}/SKILL.md`에 생성한다.

**스킬 구조:**
```
{skill-name}/
├── SKILL.md (필수) — YAML frontmatter + 본문
└── references/ (선택) — 조건부 로딩 참조 문서
```

**SKILL.md 템플릿:**

```markdown
---
name: {skill-name}
description: "{적극적 트리거 설명. 스킬이 하는 일 + 트리거 상황을 모두 기술}"
---

# {Skill Name} — {목적}

## 워크플로우
1. {단계1}
2. {단계2}

## 원칙
- {원칙1}

## 주의사항
- {주의사항}
```

**Description 작성 원칙** (중요):
- "pushy"하게 작성 — 트리거를 적극적으로 유도
- 스킬이 하는 일 + 구체적 트리거 상황 모두 기술
- "이런 경우 반드시 이 스킬을 사용할 것" 포함

스킬 작성 상세 가이드: `references/skill-writing-guide.md` 참조.

---

### Phase 5: 오케스트레이션 통합

생성된 에이전트와 스킬을 하나의 워크플로우로 연결한다.

#### 5-1. 오케스트레이터 스킬 생성

`$_SKILLS_DIR/{orchestrator-name}/SKILL.md`에 오케스트레이터를 생성한다.
오케스트레이터는 전체 팀을 조율하는 특수 스킬이다.

오케스트레이터 템플릿 상세: `references/orchestrator-template.md` 참조.

#### 5-2. 데이터 전달 프로토콜

| 전략 | 방식 | 적용 모드 |
|------|------|----------|
| 메시지 기반 | SendMessage로 직접 통신 | 팀 |
| 태스크 기반 | TaskCreate로 작업 상태 공유 | 팀 |
| 파일 기반 | `_workspace/`에 파일 쓰기/읽기 | 팀 + 서브 |
| 반환값 기반 | Agent 도구 반환 메시지 | 서브 |

#### 5-3. CLAUDE.md 포인터 등록

(선택) 프로젝트 `CLAUDE.md`가 있으면 하네스 포인터 등록:

```markdown
## 하네스: {도메인명}

**목표:** {한 줄 목표}
**트리거:** {도메인} 관련 작업 요청 시 `{orchestrator}` 스킬 사용.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 |
|------|----------|------|
```

CLAUDE.md에는 포인터(트리거 규칙 + 변경 이력)만 기록한다.
에이전트/스킬 목록은 오케스트레이터와 파일 시스템이 관리하므로 중복 금지.

---

### Phase 6: 검증

#### 6-1. 구조 검증
- [ ] 모든 에이전트 파일이 `$_AGENTS_DIR/`에 존재
- [ ] 모든 스킬 파일이 `$_SKILLS_DIR/`에 존재
- [ ] 각 SKILL.md에 name + description frontmatter 존재
- [ ] 오케스트레이터 스킬 1개 포함
- [ ] 에이전트 간 참조 일관성

#### 6-2. 트리거 검증
- 각 스킬 description이 적절한 트리거 키워드를 포함하는지 확인
- "다시 실행", "업데이트", "수정" 등 후속 작업 키워드 포함 확인

#### 6-3. 드라이런
- Phase 순서 논리적 검토
- 데이터 전달 경로에 빈 구간 없는지 확인
- 에이전트 입/출력이 이전/다음 Phase와 연결되는지 확인

---

### Phase 7: 운영/유지보수

기존 하네스 점검·수정·동기화.

#### 7-1. 현황 감사
- `$_AGENTS_DIR/` 파일 목록과 오케스트레이터 구성 비교 → 불일치 보고
- `$_SKILLS_DIR/` 디렉토리 목록과 스킬 구성 비교 → 불일치 보고

#### 7-2. 변경
사용자 요청에 따라 에이전트/스킬 추가, 수정, 삭제.

#### 7-3. 진화
실행 후 피드백을 수집하여 하네스 개선:
- 피드백 유형별 수정 대상 가이드
- 변경 이력 관리

| 피드백 유형 | 수정 대상 |
|-----------|----------|
| 결과물 품질 | 해당 에이전트의 스킬 |
| 에이전트 역할 | 에이전트 정의 `.md` |
| 워크플로우 | 오케스트레이터 스킬 |
| 트리거 누락 | 스킬 description |

---

## 산출물 체크리스트

생성 완료 후 확인:

- [ ] `$_AGENTS_DIR/{name}.md` — 에이전트 정의 파일들
- [ ] `$_SKILLS_DIR/{name}/SKILL.md` — 스킬 파일들
- [ ] `$_SKILLS_DIR/{orchestrator}/SKILL.md` — 오케스트레이터 스킬
- [ ] 각 SKILL.md에 적극적 description + 후속 작업 키워드
- [ ] Phase 순서 + 데이터 흐름 검증 완료
- [ ] 에이전트 간 참조 일관성
- [ ] (선택) CLAUDE.md에 하네스 포인터 등록

---

## 참고

- 아키텍처 패턴: `references/agent-design-patterns.md`
- 오케스트레이터 템플릿: `references/orchestrator-template.md`
- 스킬 작성 가이드: `references/skill-writing-guide.md`
- Deep Mode 방법론: `references/ouroboros-deep-mode.md`
- 에이전트 정의 템플릿: `references/harness-templates/agent-definition.md`
- 스킬 템플릿: `references/harness-templates/skill-definition.md`
- 오케스트레이터 템플릿: `references/harness-templates/orchestrator-skill.md`
