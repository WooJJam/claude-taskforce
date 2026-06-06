# 역할별 AI 가상 개발팀 (ai-dev-team)

> 요구사항 분석 → 설계 → TDD → 코드 리뷰 → QA.
> 원래 **팀이 나눠 하던 개발 과정**을 역할별 AI 에이전트로 분리하고, Orchestrator가 자동으로 연결하는 Claude Code 플러그인.

`/task` 명령 한 줄로, 혼자서도 팀처럼 검증을 거친 개발을 진행합니다.

## Table of Contents

- [Overview](#overview)
- [Motivation](#motivation)
- [Installation](#installation)
- [Usage](#usage)
  - [Approval Gate](#approval-gate)
  - [Feedback Loop](#feedback-loop)
  - [Plan Changes](#plan-changes)
- [Agents](#agents)
- [Project Structure](#project-structure)
- [Stack Support](#stack-support)
- [Output Report](#output-report)
- [Limitations](#limitations)

---

## Overview

`ai-dev-team`은 사람 개발팀의 워크플로우(요구사항 분석 · 설계 · TDD · 코드 리뷰 · QA)를 **역할별 AI 에이전트**로 분리한 Claude Code 플러그인입니다. 6개의 독립 에이전트를 Orchestrator가 순차 연결하며, 개발자는 설계 단계에서 한 번만 승인하면 나머지는 자동 진행됩니다.

**핵심 특징**

- 역할별 에이전트 분리 → 구현자와 리뷰어가 분리되어 **객관적 검증** 가능
- 단일 승인 게이트 → AI가 방향을 잘못 잡는 것을 방지
- 피드백 루프 → 리뷰·QA 실패 시 자동 재구현
- 기술 중립 설계 → 스택별 참조 파일로 다양한 프로젝트에 재사용

---

## Motivation

AI에게 "이거 만들어줘" 하면 곧장 코드가 쏟아집니다. 빠르지만, 회사에서 팀이 거치던 단계 — 요구사항 정리, 설계 합의, 코드 리뷰, QA — 가 통째로 생략됩니다. 그 단계들은 번거로워 보여도 사실 **품질을 끌어올리는 장치**입니다.

게다가 하나의 AI에게 전 과정을 맡기면 **자기가 짠 코드를 자기가 리뷰하는 셀프 리뷰**가 되어, 객관적 검토가 불가능합니다.

이 프로젝트는 그 팀 단위 과정을 역할별 AI 에이전트로 재현해, 각 역할을 독립 에이전트로 분리함으로써 이 문제를 해결합니다.

---

## Installation

```bash
/plugin marketplace add woojjam/claude-development-system
/plugin install ai-dev-team
```

설치 후 Claude Code를 재시작하면 `/task` 명령이 활성화됩니다.

> 로컬 경로에서 테스트하려면: `/plugin marketplace add /path/to/claude-development-system`

---

## Usage

```bash
/task 회원 단건 조회 API를 구현해줘
```

실행하면 다음 순서로 자동 진행됩니다.

```
Step 1. Researcher    요구사항 구체화 + 코드베이스 조사
Step 2. Planner       구현 계획 수립
─────────── ⏸ 멈춤: 개발자 승인 대기 (유일한 멈춤 지점) ───────────
Step 3. Implementer   TDD (Red → Green → Refactor) + 빌드/테스트
Step 4. Reviewer      코드 리뷰 (CRITICAL / MAJOR / MINOR 분류)
Step 5. QA            시나리오 기반 검증 (정상 / 예외 / 경계 / 회귀)
Step 6. Summarizer    최종 결과 리포트
```

### Approval Gate

Planner가 계획을 제시하면 **거기서 멈춥니다.** 계획을 검토한 뒤 아래 형식으로 승인해야 구현이 시작됩니다.

```
APPROVED: ai-run-2026-06-07-001
```

> `"ok"`, `"진행해"` 같은 모호한 표현은 승인으로 인정하지 않습니다. AI가 방향을 잘못 잡았을 때 막을 수 있도록, 명시적 승인을 강제합니다.

### Feedback Loop

Reviewer나 QA가 `재구현 필요`로 판정하면, 수정 지침과 함께 Implementer가 다시 호출됩니다. 무한 루프를 막기 위해 **재실행은 단계당 1회**로 제한됩니다.

### Plan Changes

구현 중 계획에 없던 변경이 필요해지면, 에이전트가 즉시 멈추고 `CHANGE_REQUEST:`로 사유를 보고합니다. 개발자가 계획을 수정해 재승인하면 그 지점부터 재개됩니다.

---

## Agents

| 역할 | 하는 일 | 코드 수정 |
|------|---------|:---:|
| **Researcher** | 요구사항 구체화(목표·범위·완료 조건·제약) + 코드베이스 조사 | ✗ |
| **Planner** | 구현 계획 수립(수정 대상·순서·테스트 전략·QA 시나리오·리스크) + 승인 요청 | ✗ |
| **Implementer** | TDD 풀 사이클(Red→Green→Refactor) + 빌드·테스트 검증 | ✓ |
| **Reviewer** | 아키텍처·보안·품질 리뷰, 이슈를 CRITICAL/MAJOR/MINOR로 분류 | ✗ |
| **QA** | 시나리오 검증(정상/예외/경계/회귀), 통과/실패 판정 | ✗ |
| **Summarizer** | 변경 파일·TDD 사이클·리뷰 이슈·QA 결과를 최종 리포트로 정리 | ✗ |

각 에이전트는 독립 실행되며, Orchestrator가 이전 단계의 **구조화된 결과만** 다음 에이전트에 전달합니다. 이로써 단계 격리(객관적 리뷰)와 토큰 절감을 동시에 달성합니다.

---

## Project Structure

```
claude-development-system/           ← 이 레포 = 플러그인 마켓플레이스
├── .claude-plugin/
│   └── marketplace.json             마켓플레이스 매니페스트
└── plugins/
    └── ai-dev-team/                 배포되는 플러그인
        ├── .claude-plugin/
        │   └── plugin.json          플러그인 매니페스트
        ├── commands/
        │   └── task.md              Orchestrator (전체 흐름·승인·피드백 루프)
        └── skills/
            ├── researcher/skill.md
            ├── planner/skill.md
            ├── implementer/
            │   ├── skill.md         기술 중립 TDD 원칙
            │   └── references/
            │       └── spring-boot.md
            ├── reviewer/skill.md
            ├── qa/
            │   ├── skill.md
            │   └── references/
            │       └── spring-boot.md
            └── summarizer/skill.md
```

---

## Stack Support

스킬 본문은 **특정 언어·프레임워크에 종속되지 않습니다.** TDD 원칙·리뷰 기준·QA 시나리오 같은 공통 로직만 담고, 프레임워크별 세부 컨벤션은 `references/<스택>.md`로 분리되어 있습니다.

에이전트는 프로젝트의 빌드 파일(`build.gradle`, `package.json`, `pyproject.toml` 등)로 스택을 판별하고, 해당 참조 파일이 있으면 그 컨벤션을 따릅니다. 없으면 일반 원칙을 그 스택의 표준 테스트 프레임워크에 맞게 적용합니다.

### Supported Stacks

| Stack | Language | Reference File | Status |
|-------|----------|----------------|:------:|
| Spring Boot | Java | `references/spring-boot.md` | ✅ Available |
| Node.js | JavaScript / TypeScript | — | 🚧 Planned |
| Python | Python | — | 🚧 Planned |

### Adding a New Stack

다른 스택을 쓴다면 참조 파일만 추가하면 됩니다. 예를 들어 Node.js 지원:

```
plugins/ai-dev-team/skills/implementer/references/node.md
plugins/ai-dev-team/skills/qa/references/node.md
```

해당 파일에 그 스택의 테스트 프레임워크 문법(Jest 등), 단위/통합 테스트 작성법, 빌드·검증 명령을 적어두면 됩니다.

---

## Output Report

작업이 끝나면 Summarizer가 결과 리포트를 남깁니다. 변경 파일, TDD 사이클, 리뷰 이슈(심각도별), QA 결과, 완료 체크리스트가 한 장에 정리되어 **PR 본문·회고·이력 추적**에 그대로 활용할 수 있습니다.

```
## 최종 요약 리포트 — run-id: ai-run-2026-06-07-001

### 구현 결과       (변경 파일 + TDD 사이클 표)
### 코드 리뷰 결과   (CRITICAL / MAJOR / MINOR)
### QA 결과         (시나리오별 통과/실패)
### 완료 상태       (체크리스트)
```

---

## Limitations

- 작은 기능 단위에서 주로 검증되었습니다. 여러 파일에 걸친 대규모 기능에서의 컨텍스트 전달은 더 검증이 필요합니다.
- 재실행은 단계당 1회로 제한됩니다. 복잡한 수정이 한 번에 안 끝날 수 있습니다.
- 이 시스템의 품질은 곧 **에이전트 역할 정의서(스킬)의 품질**입니다. 프로젝트 특성에 맞게 스킬을 다듬으면 결과가 좋아집니다.
