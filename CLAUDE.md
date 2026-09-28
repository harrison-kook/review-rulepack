# CLAUDE.md — review-rulepack 공통 행동 지침

이 저장소는 `ai-review-engine`이 실행 시 대상 레포의 `.claude/`로 복사해 넣는 룰팩이다.
여기 정의된 규칙은 reviewer/tester 에이전트가 반드시 따라야 하는 행동 계약이다.

## 리뷰 톤
- 근거(evidence) 없는 지적은 절대 하지 않는다. "~일 수 있습니다", "확인이 필요합니다" 같은 추측성 표현 금지.
- 규칙 ID가 없는 지적은 하지 않는다. 이 룰팩의 `rules/`에 정의되지 않은 항목은 지적 대상이 아니다.
- 한 지적당 하나의 규칙 위반만 다룬다. 여러 문제를 한 문단에 섞지 않는다.
- 스타일/포맷 문제는 결정적 도구(Checkstyle/PMD)가 이미 잡은 것은 중복 지적하지 않는다.
- 어조는 단정적이고 간결하게. "~한 것 같습니다"가 아니라 "~입니다"로 서술한다.

## 출력 형식
- 최종 출력은 **Findings JSON만** 낸다 (`schema/findings.schema.json` 준수).
- 자연어 설명, 인사말, 요약 문단을 Findings JSON 앞뒤에 덧붙이지 않는다.
- 하나의 Finding은 반드시 `ruleId`, `evidence`를 채운다. 둘 중 하나라도 비어 있으면 해당 Finding은 출력하지 않는다.
- `fingerprint`는 `sha1(ruleId + file + normalizedEvidence)`로 계산한다.

## 금지사항
- 룰팩(`rules/`, `testcases/`)에 없는 임의 규칙을 만들어 지적하지 않는다.
- 대상 레포의 시크릿, 환경변수 값, `.env` 파일 내용을 Finding의 `evidence`/`message`에 그대로 노출하지 않는다.
- diff 모드에서는 변경되지 않은 파일/라인을 지적 대상으로 삼지 않는다 (주변 코드는 컨텍스트로만 사용).
- 결정적 도구가 이미 처리 가능한 항목(포맷팅, import 순서 등)은 LLM이 재판단하지 않는다.

## 규칙 우선순위 및 병합
- 병합 순서: `common` → 스택(`java-spring` 등) → `domain` → `team` → 레포 `.review.yml`의 `overrides` (뒤가 앞을 덮어씀).
- `severity`는 레포 `overrides`에서 낮출 수 있지만, 규칙 자체를 비활성화하려면 `disable` 목록에 명시적으로 추가해야 한다.

## 참고
- 전체 설계는 `ai-review-engine`의 `docs/AI_REVIEW_SYSTEM_PLAN.md`를 따른다.
- 규칙 작성 규약은 `report-template.md` 및 각 `rules/`, `testcases/` 파일 형식을 참고한다.
