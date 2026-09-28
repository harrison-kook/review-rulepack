# review-rulepack

`ai-review-engine`이 실행 시 대상 레포의 `.claude/`로 복사해 넣는 리뷰 규칙 · 테스트케이스 룰팩 저장소.
전체 설계 배경은 `ai-review-engine`의 `docs/AI_REVIEW_SYSTEM_PLAN.md`를 참고한다.

## 구조

```
review-rulepack/
├── CLAUDE.md              # 공통 행동 지침 (톤, 출력 형식, 금지사항)
├── rules/                 # 리뷰 규칙 (common → java-spring → domain → team)
├── testcases/             # 테스트 시나리오 (tester 에이전트, 2단계용)
├── agents/                # reviewer(1단계) / tester(2단계 스텁) 서브에이전트 정의
├── commands/               # /review(1단계) / /gen-test(2단계 스텁)
├── schema/                # findings.schema.json, review-config.schema.json
└── report-template.md     # Markdown 리포트 렌더링 템플릿
```

## 버전 관리

- 버전은 Git 태그로 관리한다 (`v0.1.0`, `v0.2.0` ...).
- 각 레포는 `.review.yml`의 `rulepack: our-org/review-rulepack@vX.Y.Z`로 태그를 고정해서 참조한다.
- 규칙을 수정해도 기존에 태그를 고정한 레포의 동작은 바뀌지 않는다. 새 태그를 만들고 각 레포가 명시적으로 올려야 한다.

## 규칙 ID 접두어

`SEC`(보안) · `ERR`(예외처리) · `LAY`(계층) · `JPA`(영속성) · `PAY`(결제) · `STYLE`(스타일) · `TEAM`(팀 규칙)

## 규칙/테스트케이스 작성 규약

- 규칙: `AI_REVIEW_SYSTEM_PLAN.md` 5.1절 형식 (ID, 심각도, 검사 방식, 판단 기준, Bad/Good 예시, 예외)을 따른다.
- 테스트케이스: 5.2절 형식 (ID, 대상 계층, 우선순위, 탐색 힌트, Given/When/Then)을 따른다.
- 새 규칙은 반드시 ID를 붙인다 (위반 집계, 오탐 추적을 위해 필수).

## 현재 단계

로드맵 1단계: `rules/`, `agents/reviewer.md`, `commands/review.md`, `schema/` 구성 완료.
`agents/tester.md`, `commands/gen-test.md`는 2단계(커버리지 게이트) 작업 시 구체화할 스텁 상태다.
