# 스킬 작성 가이드

## 개요

Skills are procedural knowledge + tool bundles that agents use to perform work. This guide covers how to write effective skills.

잘 작성된 스킬은 에이전트가 특정 작업을 일관된 품질로 수행하게 만든다. 반대로 잘못 작성된 스킬은 무시되거나, 잘못된 맥락에서 트리거되어 오히려 품질을 떨어뜨린다.

## 스킬 구조

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown body
└── Bundled Resources (optional)
    ├── scripts/    - executable code for repetitive tasks
    ├── references/ - reference documents loaded conditionally
    └── assets/     - files used in output (templates, images)
```

SKILL.md는 단일 진입점이다. 모든 핵심 지침은 이 파일에 담고, 분량이 커질 경우 `references/`로 분리한다. `scripts/`에는 에이전트가 실행할 수 있는 셸 스크립트나 코드 템플릿을 넣는다. `assets/`은 출력물 생성에 필요한 템플릿이나 이미지다.

## Description 작성 — 적극적 트리거 유도

Description은 스킬이 언제 트리거될지를 결정한다. 에이전트는 description을 보고 "지금 이 스킬을 써야 하는가?"를 판단한다.

**좋은 예:**
```yaml
description: >
  Build production-grade React Native UIs with high design quality.
  Use when building components, pages, or any UI work.
  Triggers on design, layout, styling, animation tasks.
```

**나쁜 예:**
```yaml
description: "React Native UI 컴포넌트를 만듭니다."
```

좋은 예는 **트리거 키워드**(build, design, layout, styling, animation)를 명시적으로 나열해서 에이전트가 매칭하기 쉽게 만든다. Description이 "pushy"해야 한다 — Claude는 스킬 트리거에 보수적이므로, 약간 과하게 트리거되는 편이 놓치는 것보다 낫다.

**트리거 키워드 포함 전략:**
- 초기 요청 키워드: "create", "build", "make", "implement", "add"
- 후속 요청 키워드: "refactor", "fix", "update", "change", "modify"
- 도메인 키워드: "design", "layout", "animation", "component", "page"
- 컨텍스트 키워드: "UI", "screen", "navigation", "style", "theme"

## 본문 작성 원칙

**Why를 설명하라:** 에이전트가 LLM 기반으로 추론한다는 점을 기억하라. "이렇게 하라"는 지시만 있으면 에이전트가 판단 없이 기계적으로 따른다. "왜 이렇게 해야 하는지"를 설명하면 에이전트가 맥락에 맞게 지침을 적용할 수 있다.

**Lean하게 유지:** SKILL.md는 500줄 미만으로 유지하라. 분량이 커지면 `references/`로 분리하고, 본문에서는 "왜"와 "무엇을"만 남기고 "어떻게"는 references/에 위임한다.

**일반화하라:** 특정 예시 하나만 설명하지 말고 원칙을 설명하라. "버튼은 theme.colors.primary를 사용한다"보다 "컴포넌트는 theme 토큰을 통해 색상을 참조하고, 하드코딩하지 않는다"가 더 좋다. 예시는 원칙을 보강하는 용도로만 사용한다.

**명령형으로 작성:** "해야 한다", "하는 것이 좋다"는 표현을 피하고 "하라", "사용하라", "유지하라" 같은 명령형을 사용하라. 에이전트는 명확한 지시에 더 잘 따른다.

**Progressive Disclosure (3단계 로딩):**
1. **Metadata (YAML frontmatter)** — name + description. 에이전트가 트리거 여부를 판단하는 최소 정보.
2. **SKILL.md 본문** — 핵심 지침. 트리거된 후 로드된다.
3. **references/ (선택적)** — 상세 내용. SKILL.md에서 필요할 때만 참조하도록 지시한다.

## 스킬-에이전트 연결 방식

에이전트가 스킬을 사용하는 세 가지 방식:

1. **Skill 도구 호출 (독립 워크플로우):** 에이전트가 `skill()` 도구로 스킬을 로드한다. 가장 일반적인 방식. SKILL.md 전체가 컨텍스트에 주입된다.

2. **프롬프트 내 인라인 (짧은 지침):** 스킬 내용이 짧을 경우(< 50줄) 에이전트 시스템 프롬프트에 직접 포함시킨다. 빠른 트리거가 필요하거나 항상 적용되어야 하는 규칙에 적합하다.

3. **레퍼런스 로드 (조건부 대용량 콘텐츠):** SKILL.md에서 "X 작업을 할 때는 references/x.md를 읽으라"고 지시한다. 에이전트가 필요한 시점에만 추가 로드하므로 컨텍스트를 절약할 수 있다.

## OpenCode 스킬 형식

OpenCode 스킬은 YAML frontmatter를 사용한다:

```yaml
---
name: skill-name
description: "Trigger description. Use when... Triggers on..."
---
```

**필수 필드:** `name`, `description`

**권장 필드:** 없음. 모든 추가 정보는 본문 마크다운에 작성한다.

## 검증 체크리스트

- [ ] `name` + `description`이 YAML frontmatter에 존재하는가?
- [ ] Description이 "pushy"하며 트리거 키워드를 포함하는가?
- [ ] SKILL.md가 500줄 미만인가? 아니라면 `references/`로 분리했는가?
- [ ] 명령형 어조로 작성되었는가? ("하라", "사용하라")
- [ ] Why가 설명되었는가? (무엇만이 아니라 왜를 함께 설명)
- [ ] `references/`가 300줄 이상이면 목차(Table of Contents)가 있는가?
- [ ] 예시가 특정 구현에 과도하게 의존하지 않고 일반화되었는가?
- [ ] Progressive Disclosure 원칙을 따르는가? (SKILL.md → references/)
