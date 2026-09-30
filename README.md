# review-rulepack

`ai-review-engine`이 실행 시 대상 레포의 `.claude/`로 복사해 넣는 리뷰 규칙 · 테스트케이스 룰팩 저장소.
전체 설계 배경은 `ai-review-engine`의 `docs/AI_REVIEW_SYSTEM_PLAN.md`를 참고한다.

## 구조

```
review-rulepack/
├── CLAUDE.md              # 공통 행동 지침 (톤, 출력 형식, 금지사항)
├── rules/                 # 리뷰 규칙 (common → java-spring → domain → team)
├── testcases/             # 테스트 시나리오 (tester 에이전트가 소비)
├── agents/                # reviewer / tester 서브에이전트 정의
├── commands/               # /review / /gen-test
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

## 팀 규칙 설정하기

팀 고유 컨벤션(네이밍 규칙, 내부 정책 등)은 `rules/team/our-team.md`에 작성한다. 다른 규칙과
형식은 동일하다 — `TEAM-XXX` 접두어로 순번을 이어서 추가한다.

```markdown
## TEAM-002: 클래스명은 도메인+역할 접미사를 따른다 (예: XxxService, XxxRepository)
- 심각도: LOW
- 검사 방식: LLM
- 판단 기준: 서비스/리포지토리 계층 클래스명이 역할을 드러내는 접미사 없이 명명됨
- Bad 예시:
  ```java
  public class UserLogic { ... }
  ```
- Good 예시:
  ```java
  public class UserService { ... }
  ```
- 예외: 유틸리티/헬퍼성 클래스(XxxUtils, XxxHelper)는 제외
```

### 대상 레포에서 team 규칙 켜기

레포의 `.review.yml`의 `profiles`에 `team`을 넣어야 실제로 적용된다. 안 넣으면 규칙 파일이
있어도 무시된다.

```yaml
profiles: [common, java-spring, domain, team]
```

### 특정 규칙만 끄거나 심각도 바꾸기 (레포별 override)

전체 팀에 적용된 규칙을 특정 레포에서만 예외 처리하고 싶으면 `.review.yml`의 `overrides`를
쓴다. `disable`은 완전히 끄고, 규칙 ID를 키로 쓰면 심각도만 바꾼다.

```yaml
overrides:
  disable: [TEAM-001]           # TEAM-001은 이 레포에서 검사하지 않음
  TEAM-002: { severity: MEDIUM } # TEAM-002는 LOW 대신 MEDIUM으로 취급
```

### 여러 팀이 서로 다른 컨벤션을 쓸 때

`rules/team/`은 디렉터리 하나뿐이라 팀마다 컨벤션이 다르면 그대로 쓸 수 없다. 이 경우
`rules/team-backend/`, `rules/team-admin/`처럼 프로필 디렉터리를 팀별로 나누고(`testcases/`도
동일하게), 각 레포의 `.review.yml`에서 해당 팀 프로필만 골라 쓴다.

```yaml
# 팀 A 레포
profiles: [common, java-spring, domain, team-backend]
# 팀 B 레포
profiles: [common, java-spring, domain, team-admin]
```

### 규칙 추가/수정 후 배포하기

규칙을 고치거나 추가해도 이미 특정 태그(`v0.2.0` 등)를 고정 참조 중인 레포의 동작은 바뀌지
않는다 — 새 태그를 만들고, 각 레포의 `.review.yml`에 있는 `rulepack: harrison-kook/review-rulepack@vX.Y.Z`를
명시적으로 올려줘야 반영된다. 접근 권한(팀 전체 write vs 승인제)은 설계서 13장 미결 사항 결정을
참고한다.

## 현재 단계

로드맵 1단계(리뷰) + 2단계(테스트 생성) 구성 완료:
- `rules/`, `agents/reviewer.md`, `commands/review.md`, `schema/` — 소스 리뷰
- `testcases/`, `agents/tester.md`, `commands/gen-test.md` — 테스트 생성. tester는 테스트
  코드만 작성하고 실행하지 않는다(`Bash` 도구 없음) — 실행/판정은 엔진의 `StackAdapter.test()`가
  결정적으로 수행한다 (`ai-review-engine`의 `ReviewCommand.Phase.TESTRUN`).
