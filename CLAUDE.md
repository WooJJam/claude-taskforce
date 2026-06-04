# AI 개발 워크플로우

이 프로젝트는 `ai-development-workflow.md`에 정의된 AI 가상 개발팀 워크플로우를 따른다.

---

## 워크플로우 적용 조건

**`/task` 커맨드로 명시적으로 요청한 경우에만** 아래 워크플로우를 적용한다.
그 외 일반 요청은 워크플로우 없이 바로 처리한다.

---

## 워크플로우 흐름 (`/task` 실행 시)

```
/task 실행
  → [자동] Step 1: Researcher   요구사항 정리 + 코드베이스 조사
  → [자동] Step 2: Planner      구현 계획 수립
  → [멈춤] APPROVED: <run-id>   ← 유일한 멈춤 지점

APPROVED 수신 시:
  → [자동] Step 3: Implementer  TDD (Red→Green→Refactor) + 빌드/테스트 검증
  → [자동] Step 4: Reviewer     코드 리뷰 (아키텍처, 보안, 코드 품질)
  → [자동] Step 5: QA           시나리오 기반 검증 결과 출력
  → [자동] Step 6               완료 확인 + 최종 요약
```

## 에이전트 스킬 위치

```
.claude/skills/
  researcher/skill.md    요구사항 정리 + 코드베이스 조사
  planner/skill.md       구현 계획 수립
  implementer/skill.md   TDD 풀 사이클 + 빌드/테스트 검증
  reviewer/skill.md      코드 리뷰 (아키텍처, 보안, 코드 품질)
  qa/skill.md            시나리오 기반 QA 검증
```

---

## 핵심 규칙

- `APPROVED: <run-id>` 없이 Step 3 이후 진행 금지
- 승인 형식: `APPROVED: ai-run-YYYY-MM-DD-NNN`
- "ok", "진행해" 등 모호한 표현은 승인으로 인정하지 않는다
- 계획 외 파일 변경 시: 즉시 중단 → `CHANGE_REQUEST:` 기록 → 재승인

---

## 완료 기준

- [ ] 승인된 계획 기준으로 구현
- [ ] 전체 테스트 통과
- [ ] 코드 리뷰 결과 기록
- [ ] QA 시나리오 검증 결과 기록
- [ ] 최종 변경 요약 작성
