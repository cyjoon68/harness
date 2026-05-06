<p align="center">
  <img src="https://img.shields.io/badge/Version-1.0.0-brightgreen.svg" alt="Version">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License"></a>
  <img src="https://img.shields.io/badge/OpenCode-Skill-purple.svg" alt="OpenCode Skill">
  <img src="https://img.shields.io/badge/Patterns-6_Architectures-orange.svg" alt="6 Architecture Patterns">
</p>

# Harness — OpenCode Agent Team & Skill Factory

> **도메인 설명 한 줄로 전문 에이전트 팀과 스킬을 자동 생성한다.**

**"하네스 구성해줘"** — 이 한 문장으로 프로젝트에 맞는 에이전트 팀을 설계하고, 각 에이전트가 사용할 스킬을 생성하는 메타 스킬.

gstack 스타일의 글로벌 스킬로 동작하며, 생성된 하네스는 프로젝트 로컬(`.agents/`)에 구성된다.

---

## 레이어

Harness는 OpenCode 스킬 생태계의 **L3 Meta-Factory** 층에 위치한다.

| 층위 | 하는 일 |
|------|---------|
| **L3 — Meta-Factory / Team-Architecture Factory** (이 프로젝트) | 도메인 설명 → 에이전트 팀 + 스킬, 6가지 사전 정의된 팀 패턴 |
| L2 — Cross-Harness Workflow | 여러 하네스의 스킬/규칙 표준화 |
| L1 — Project Skills | 프로젝트별 에이전트 정의와 스킬 |
| L0 — Built-in Skills | OpenCode 기본 제공 스킬 |

---

## 핵심 기능

- **에이전트 팀 설계** — 6가지 아키텍처 패턴 (파이프라인, 팬아웃/팬인, 전문가 풀, 생성-검증, 감독자, 계층적 위임)
- **스킬 자동 생성** — Progressive Disclosure 패턴의 스킬 파일 생성
- **Deep Mode** — Ouroboros 방법론 기반 심층 도메인 분석 (소크라테스식 질문 → 모호성 점수 → 명세)
- **진화 메커니즘** — 사용 피드백을 반영하여 지속적 개선
- **gstack 스타일** — 글로벌 설치, 프로젝트 로컬 산출물

---

## 설치

### 요구사항
- [OpenCode](https://opencode.ai/)

### 직접 설치

```bash
# 저장소 클론
git clone --single-branch --depth 1 https://github.com/{your-username}/harness.git ~/.config/opencode/skills/harness
```

### setup 스크립트 사용

```bash
git clone https://github.com/{your-username}/harness.git
cd harness
chmod +x setup && ./setup
```

---

## 사용법

OpenCode에서 다음과 같이 트리거한다:

### Standard 모드

```
하네스 구성해줘
에이전트 팀 만들어줘
이 프로젝트에 맞는 전문가 팀 구성해줘
```

### Deep Mode (Ouroboros 인터뷰)

```
딥 모드로 하네스 구성해줘
인터뷰하면서 하네스 구성해줘
명세 기반으로 하네스 설계해줘
```

Deep Mode는 다음 과정을 거친다:
1. **소크라테스식 인터뷰** — 5-7개의 질문으로 숨겨진 요구사항 발굴
2. **모호성 점수 산정** — 목표/제약/성공 기준의 명확도를 수치화 (Ambiguity ≤ 0.2 목표)
3. **Seed 명세 생성** — 확정된 스펙으로 정리
4. **실행** — 표준 하네스 생성 워크플로우

### 유지보수

```
하네스 점검해줘
하네스에 에이전트 추가해줘
하네스 업데이트해줘
```

---

## 워크플로우

```
Phase 0: 현황 감사 (기존 하네스 확인)
   │
   ├── Deep Mode → Phase 0-D: Ouroboros 인터뷰
   │                  │
   │                  ▼
   │              Seed 명세
   │
   ▼
Phase 1: 도메인 분석
   ▼
Phase 2: 팀 아키텍처 설계 (6 patterns)
   ▼
Phase 3: 에이전트 정의 생성 (.agents/)
   ▼
Phase 4: 스킬 생성 (.agents/skills/)
   ▼
Phase 5: 오케스트레이션 통합
   ▼
Phase 6: 검증
   ▼
Phase 7: 운영/유지보수
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

**모호성 점수**로 진행 여부를 결정한다:

```
Ambiguity = 1 − Σ(clarityᵢ × weightᵢ)
Ambiguity ≤ 0.2 → Seed 생성 가능
```

---

## 프로젝트 구조

```
harness/
├── SKILL.md                          # 메인 스킬 정의 (전체 워크플로우)
├── setup                             # 설치 스크립트 (gstack 스타일)
├── README.md                         # 이 문서
├── LICENSE
└── references/
    ├── agent-design-patterns.md      # 6가지 아키텍처 패턴
    ├── orchestrator-template.md      # 오케스트레이터 템플릿
    ├── skill-writing-guide.md        # 스킬 작성 가이드
    ├── ouroboros-deep-mode.md        # Deep Mode 방법론
    └── harness-templates/            # 생성 산출물 템플릿
        ├── agent-definition.md       # 에이전트 정의 템플릿
        ├── skill-definition.md       # 스킬 템플릿
        └── orchestrator-skill.md     # 오케스트레이터 템플릿
```

## 생성되는 산출물

하네스를 실행하면 프로젝트에 다음 파일들이 생성된다:

```
your-project/
├── .agents/
│   ├── {agent-1}.md              # 에이전트 정의
│   ├── {agent-2}.md
│   └── {agent-3}.md
├── .agents/
│   └── skills/
│       ├── {skill-1}/SKILL.md    # 스킬 파일
│       ├── {skill-2}/SKILL.md
│       └── {orchestrator}/SKILL.md  # 오케스트레이터
└── CLAUDE.md (optional)          # 하네스 포인터
```

---

## 사용 사례

### 리서치 하네스
```
리서치 하네스를 구성해줘. 어떤 주제든 다각도로 조사하는 팀이 필요해.
웹 검색, 학술 자료, 커뮤니티 분석을 병렬로 수행하고 결과를 종합하는 팀.
```

### 코드 리뷰 하네스
```
코드 리뷰 하네스를 구성해줘. 아키텍처, 보안, 성능, 스타일을
병렬 검사하고 통합 리포트를 생성하는 팀.
```

### 콘텐츠 제작 하네스
```
웹툰 제작 하네스를 구성해줘. 스토리, 캐릭터, 패널 레이아웃,
대사를 생성-검증 파이프라인으로 처리하는 팀.
```

---

## 라이선스

MIT

---

*Built with inspiration from [revfactory/harness](https://github.com/revfactory/harness), [garrytan/gstack](https://github.com/garrytan/gstack), and [Q00/ouroboros](https://github.com/Q00/ouroboros).*
