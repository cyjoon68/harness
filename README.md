<p align="center">
  <img src="https://img.shields.io/badge/Version-1.0.0-brightgreen.svg" alt="Version">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License"></a>
  <img src="https://img.shields.io/badge/Patterns-6_Architectures-orange.svg" alt="6 Architecture Patterns">
  <img src="https://img.shields.io/badge/Deep_Mode-Ouroboros-purple.svg" alt="Deep Mode">
</p>

# Harness-Creator — Agent Team & Skill Factory

> **한 문장으로 전문 에이전트 팀과 스킬을 자동 생성한다.**

**"하네스 구성해줘"** — 이 한 문장으로 프로젝트에 맞는 에이전트 팀을 설계하고, 각 에이전트가 사용할 스킬을 생성하는 메타 스킬.

gstack 스타일로 여러 AI 코딩 에이전트(Claude Code, Codex CLI, OpenCode, Cursor 등)에서 동작하며,
생성된 하네스는 프로젝트 로컬(`.agents/`)에 구성된다.
**하네스 구축 시에는 항상 Deep Mode(소크라테스식 인터뷰 → 모호성 점수 → Seed 명세)를 거친다.**

---

## 레이어

| 층위 | 하는 일 |
|------|---------|
| **L3 — Meta-Factory / Team-Architecture Factory** (이 프로젝트) | 도메인 설명 → 에이전트 팀 + 스킬, 6가지 팀 패턴 |
| L2 — Cross-Harness Workflow | 여러 하네스의 스킬/규칙 표준화 |
| L1 — Project Skills (생성된 하네스) | 프로젝트별 에이전트 정의와 스킬 |

---

## 핵심 기능

- **에이전트 팀 설계** — 6가지 아키텍처 패턴 (파이프라인, 팬아웃/팬인, 전문가 풀, 생성-검증, 감독자, 계층적 위임)
- **스킬 자동 생성** — Progressive Disclosure 패턴의 스킬 파일 생성
- **Deep Mode 기본** — Ouroboros 방법론 기반 심층 도메인 분석 (소크라테스식 질문 → 모호성 점수 → Seed 명세 → 생성)
- **멀티 플랫폼** — Claude Code, Codex CLI, OpenCode, Cursor, Factory Droid, Kiro CLI 지원
- **진화 메커니즘** — 사용 피드백을 반영하여 지속적 개선

---

## 설치

### 요구사항
지원 AI 코딩 에이전트 중 하나가 설치되어 있어야 함.

### 자동 설치 (권장)

```bash
git clone --single-branch --depth 1 https://github.com/cyjoon68/harness.git
cd harness
chmod +x setup && ./setup
```

설치 스크립트가 시스템에 설치된 AI 코딩 에이전트를 자동 감지하여 스킬을 등록한다.

### 특정 플랫폼 지정

```bash
# OpenCode 전용
./setup --host opencode

# Claude Code 전용
./setup --host claude

# Codex CLI 전용
./setup --host codex

# Cursor 전용
./setup --host cursor
```

지원: `claude`, `codex`, `opencode`, `cursor`, `factory`, `kiro`, `auto` (기본값)

### 수동 설치

```bash
# OpenCode
git clone --single-branch --depth 1 https://github.com/cyjoon68/harness.git ~/.config/opencode/skills/harness-creator

# Claude Code
git clone --single-branch --depth 1 https://github.com/cyjoon68/harness.git ~/.claude/skills/harness-creator

# Codex CLI
git clone --single-branch --depth 1 https://github.com/cyjoon68/harness.git ~/.codex/skills/harness-creator
```

### 팀 모드

팀 저장소에서 모든 구성원이 하네스 크리에이터를 사용하게 하려면:

```bash
./setup --team
```

`.gitignore`에 `.agents/`가 추가되고, 팀원 각자가 `./setup`을 실행하면 된다.

---

## 사용법

AI 코딩 에이전트에서 자연어로 트리거:

```
하네스 구성해줘
에이전트 팀 만들어줘
이 프로젝트에 맞는 전문가 팀 구성해줘
```

하네스 구축 시 항상 딥 모드 과정을 거친다:
1. **소크라테스식 인터뷰** — 5-7개의 질문으로 숨겨진 요구사항 발굴
2. **모호성 점수 산정** — 목표/제약/성공 기준의 명확도를 수치화 (Ambiguity ≤ 0.2 목표)
3. **Seed 명세 생성** — 확정된 스펙으로 정리, 사용자 확인
4. **하네스 생성** — 명세 기반 에이전트 팀 및 스킬 생성

생성된 하네스는 별도 트리거로 실행:
```
{도메인} 작업 시작해줘
{도메인} 리서치 실행해줘
```

---

## Deep Mode 적용 범위

| 단계 | Deep Mode |
|------|:---------:|
| **하네스 구축** (이 메타 스킬) | **필수** — 인터뷰 → Seed → 생성 |
| **생성된 하네스 실행** | **선택** — 작업 모호 시 사용자에게 제안 |

---

## 워크플로우

```
메타 스킬 (하네스 구축 시):
┌──────────────────────────────────────────┐
│  Phase 0: 현황 감사                      │
│       ▼                                  │
│  Phase 1: Deep Mode 인터뷰 (필수)         │
│    ├── 소크라테스식 질문                 │
│    ├── 모호성 점수 (Ambiguity ≤ 0.2)     │
│    └── Seed 명세 확정                    │
│       ▼                                  │
│  Phase 2-6: 하네스 생성                  │
│  Phase 7: 운영/유지보수                  │
└──────────────────────────────────────────┘

생성된 하네스 (작업 실행 시):
┌──────────────────────────────────────────────┐
│  Phase 0: 컨텍스트 확인                      │
│       │                                      │
│       ├─ 작업 모호? ─→ "Deep Mode 사용할까요?"│
│       │                    ├─ Yes → Phase D  │
│       │                    └─ No  → Phase 1  │
│       └─ 명확함 → 바로 Phase 1              │
│       ▼                                      │
│  Phase D: Deep Mode 인터뷰 (선택)             │
│  Phase 1-N: 도메인 작업 실행                 │
└──────────────────────────────────────────────┘
```

---

## 아키텍처 패턴

| 패턴 | 설명 | 예시 |
|------|------|------|
| **파이프라인** | 순차 의존 작업 | 문서 생성, 데이터 처리 |
| **팬아웃/팬인** | 병렬 독립 작업 → 통합 | 리서치, 코드 리뷰 |
| **전문가 풀** | 상황별 선택 호출 | 문의 처리, 분류 |
| **생성-검증** | 생성 후 품질 검수 | 콘텐츠 제작, 번역 |
| **감독자** | 중앙 에이전트가 동적 분배 | 마이그레이션 |
| **계층적 위임** | 상위→하위 재귀적 위임 | 풀스택 개발 |

---

## Deep Mode — Ouroboros 방법론

Ouroboros의 철학을 차용한 **명세 우선 접근법**.

```
    ◇ Wonder         ◇ 설계
   ╱  (넓히기)       ╱  (넓히기)
  ╱    탐색         ╱    창조
 ╱                 ╱
◆ ──────────── ◆ ──────────── ◆
 ╲                 ╲
  ╲    정의         ╲    전달
   ╲  (좁히기)      ╲  (좁히기)
    ◇ 온톨로지       ◇ 평가
```

첫 번째 다이아몬드: 질문을 넓히고 본질로 좁힌다.
두 번째 다이아몬드: 설계를 넓히고 검증된 산출물로 좁힌다.

**모호성 점수**가 임계값을 통과해야만 하네스 생성 진행:

```
Ambiguity = 1 − Σ(clarityᵢ × weightᵢ)
Ambiguity ≤ 0.2 → ✅ Seed 생성 → 하네스 구축 진행
Ambiguity > 0.2  → 🔄 추가 질문으로 모호성 제거 후 재측정
```

---

## 프로젝트 구조

```
harness-creator/
├── SKILL.md                          # 메인 스킬 정의 (전체 워크플로우)
├── setup                             # 설치 스크립트 (멀티 플랫폼)
├── README.md                         # 이 문서
└── references/
    ├── agent-design-patterns.md      # 6가지 아키텍처 패턴
    ├── orchestrator-template.md      # 오케스트레이터 템플릿
    ├── skill-writing-guide.md        # 스킬 작성 가이드
    ├── ouroboros-deep-mode.md        # Deep Mode 방법론
    └── harness-templates/            # 생성 산출물 템플릿
        ├── agent-definition.md       # 에이전트 정의 템플릿
        ├── orchestrator-skill.md     # 오케스트레이터 템플릿 (Deep Mode 선택)
        └── skill-definition.md       # 스킬 템플릿
```

---

## 사용 사례

**리서치 하네스**
```
리서치 하네스를 구성해줘. 어떤 주제든 다각도로 조사하는 팀이 필요해.
웹 검색, 학술 자료, 커뮤니티 분석을 병렬로 수행하고 결과를 종합하는 팀.
```

**코드 리뷰 하네스**
```
코드 리뷰 하네스를 구성해줘. 아키텍처, 보안, 성능, 스타일을
병렬 검사하고 통합 리포트를 생성하는 팀.
```

**콘텐츠 제작 하네스**
```
웹툰 제작 하네스를 구성해줘. 스토리, 캐릭터, 패널 레이아웃,
대사를 생성-검증 파이프라인으로 처리하는 팀.
```

---

## 라이선스

MIT

---

*Built with inspiration from [revfactory/harness](https://github.com/revfactory/harness), [garrytan/gstack](https://github.com/garrytan/gstack), and [Q00/ouroboros](https://github.com/Q00/ouroboros).*
