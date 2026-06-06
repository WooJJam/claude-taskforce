# /task — AI 개발 워크플로우 오케스트레이터

사용자 요청을 워크플로우로 처리한다.
각 단계는 독립된 서브 에이전트로 실행하며, **이전 단계의 결과를 다음 에이전트 프롬프트에 명시적으로 포함**시킨다.

## 스킬 위치

| 단계 | 역할 | 스킬 파일 |
|------|------|----------|
| Step 1 | Researcher | `${CLAUDE_PLUGIN_ROOT}/skills/researcher/skill.md` |
| Step 2 | Planner | `${CLAUDE_PLUGIN_ROOT}/skills/planner/skill.md` |
| Step 3 | Implementer | `${CLAUDE_PLUGIN_ROOT}/skills/implementer/skill.md` |
| Step 4 | Reviewer | `${CLAUDE_PLUGIN_ROOT}/skills/reviewer/skill.md` |
| Step 5 | QA | `${CLAUDE_PLUGIN_ROOT}/skills/qa/skill.md` |
| Step 6 | Summarizer | `${CLAUDE_PLUGIN_ROOT}/skills/summarizer/skill.md` |

## 전체 흐름

```
[자동] Step 1: Researcher   → 결과(A) 수신
[자동] Step 2: Planner      → 결과(B: 구현 계획) 수신, 컨텍스트: A 포함
[멈춤] APPROVED: <run-id> 대기
[자동] Step 3: Implementer  → 결과(C) 수신, 컨텍스트: B 포함
[자동] Step 4: Reviewer     → 결과 수신, 컨텍스트: C 포함
         ├─ CRITICAL? → Step 3 재실행 (Fix 모드, max 1회)
         └─ OK        → Step 5
[자동] Step 5: QA           → 결과 수신, 컨텍스트: B + C 포함
         ├─ 실패?     → Step 3 재실행 (Fix 모드, max 1회)
         └─ 통과      → Step 6
[자동] Step 6: 완료 확인 + 최종 요약
```

## 핵심 규칙

- `APPROVED: <run-id>` 없이 Step 3 이후 진행 금지
- 승인 형식: `APPROVED: ai-run-YYYY-MM-DD-NNN`
- `CHANGE_REQUEST:` 발생 시 즉시 사용자에게 전달하고 재승인 대기
- `CHANGE_REQUEST:` 재승인 시 run-id는 기존과 동일하게 유지
- Implementer 재실행은 Reviewer/QA당 최대 1회

---

## 실행 지침

### Step 1 — Researcher

Agent 도구로 서브 에이전트를 실행한다.

프롬프트:
```
`${CLAUDE_PLUGIN_ROOT}/skills/researcher/skill.md`를 읽고 그 내용에 따라 Researcher 역할을 수행하라.

[사용자 요구사항]
{사용자가 입력한 요구사항 그대로}

[프로젝트 경로]
{현재 프로젝트 절대 경로}
```

에이전트 결과를 **결과(A)** 로 보관한다.

---

### Step 2 — Planner

Agent 도구로 서브 에이전트를 실행한다.

프롬프트:
```
`${CLAUDE_PLUGIN_ROOT}/skills/planner/skill.md`를 읽고 그 내용에 따라 Planner 역할을 수행하라.

[Researcher 조사 결과]
{결과(A) 전문}
```

에이전트 결과(구현 계획)를 **결과(B)** 로 보관한다.
결과(B)를 사용자에게 제시하고 `APPROVED: <run-id>` 대기.

**CHANGE_REQUEST 감지 시**: 사용자에게 즉시 알리고 재승인 요청.

---

### Step 3 — Implementer (APPROVED 수신 후)

Agent 도구로 서브 에이전트를 실행한다.

**신규 구현 (기본):**
```
`${CLAUDE_PLUGIN_ROOT}/skills/implementer/skill.md`를 읽고 그 내용에 따라 TDD 사이클로 구현하라.

[승인된 구현 계획]
{결과(B) 전문}

[프로젝트 경로]
{현재 프로젝트 절대 경로}
```

**재실행 (Reviewer 또는 QA 수정 요청 시):**
```
`${CLAUDE_PLUGIN_ROOT}/skills/implementer/skill.md`를 읽고 그 내용에 따라 수행하라.
수정 지침에 해당하는 범위만 수정한다. 전체 재구현하지 않는다.

[수정 지침]
{Reviewer 또는 QA의 수정 지침}

[승인된 구현 계획]
{결과(B) 전문}

[프로젝트 경로]
{현재 프로젝트 절대 경로}
```

에이전트 결과를 **결과(C)** 로 보관한다.

**CHANGE_REQUEST 감지 시**: 즉시 중단하고 사용자에게 알린 뒤 재승인 대기.

---

### Step 4 — Reviewer

Agent 도구로 서브 에이전트를 실행한다.

```
`${CLAUDE_PLUGIN_ROOT}/skills/reviewer/skill.md`를 읽고 그 내용에 따라 코드 리뷰를 수행하라.

[구현 결과]
{결과(C) 전문}

[구현 목표]
{결과(B)에서 "구현 목표" 항목}
```

결과에서 **`재구현 필요: YES`** 확인 시:
- 수정 지침을 포함하여 Step 3 Fix 모드로 재실행 (1회에 한함)
- 재실행 후 Step 5로 진행

**`재구현 필요: NO`** 시 Step 5로 진행.

---

### Step 5 — QA

Agent 도구로 서브 에이전트를 실행한다.

```
`${CLAUDE_PLUGIN_ROOT}/skills/qa/skill.md`를 읽고 그 내용에 따라 QA 시나리오를 검증하라.

[QA 시나리오]
{결과(B)에서 "QA 시나리오" 항목}

[구현 결과]
{결과(C) 전문}

[프로젝트 경로]
{현재 프로젝트 절대 경로}
```

결과에서 **`재구현 필요: YES`** 확인 시:
- 수정 지침을 포함하여 Step 3 Fix 모드로 재실행 (1회에 한함)
- 재실행 후 Step 6으로 진행

**`재구현 필요: NO`** 시 Step 6으로 진행.

---

### Step 6 — Summarizer

Agent 도구로 서브 에이전트를 실행한다.
각 에이전트 결과의 전체를 넘기지 않고 **구조화된 섹션만 추출**하여 전달한다.

```
`${CLAUDE_PLUGIN_ROOT}/skills/summarizer/skill.md`를 읽고 최종 리포트를 작성하라.

[run-id]
{run-id}

[구현 결과 요약]
{결과(C)에서 [구현 결과] 섹션만 추출하여 그대로 붙여넣기}

[코드 리뷰 결과]
{Reviewer 결과에서 [CRITICAL 이슈] + [MAJOR 이슈] + [MINOR 권고 사항] 섹션만 추출하여 그대로 붙여넣기}

[QA 검증 결과]
{QA 결과에서 [QA 검증 결과] 테이블 + [종합] 섹션만 추출하여 그대로 붙여넣기}
```

---

## CHANGE_REQUEST 처리

어느 단계에서든 `CHANGE_REQUEST:` 가 발생하면:

1. 즉시 해당 에이전트를 중단시킨다
2. 사용자에게 다음 형식으로 전달한다:

```
[CHANGE_REQUEST 발생]
단계: Step N
내용: {CHANGE_REQUEST 내용}

계획 수정 후 아래 형식으로 승인해 주세요:
APPROVED: {run-id}
```

3. 새 `APPROVED: <run-id>` 수신 후 해당 단계부터 재개한다.
