---
description: 룰팩 기준으로 대상 레포(또는 diff)를 리뷰하고 Findings JSON을 출력
---

## /review

`reviewer` 서브에이전트(`agents/reviewer.md`)를 호출해 코드 리뷰를 수행합니다.

### 사전 조건
- `.review.yml`이 실행 컨텍스트(엔진)에 의해 이미 로드되어 `profiles`, `mode`, `include`/`exclude`, `overrides`, `limits`가 확정되어 있다.
- 결정적 분석(빌드/Checkstyle/PMD/SpotBugs/기존 테스트) 결과가 이미 생성되어 컨텍스트로 제공된다.
- 적용할 규칙 파일(`rules/**/*.md`)이 `profiles`에 따라 이미 이 워크스페이스의 `.claude/`에 배치되어 있다.

### 실행
1. `agents/reviewer.md`의 지침에 따라 규칙을 로드하고 병합한다 (common → 스택 → domain → team → overrides).
2. `mode`가 `diff`면 변경 파일/라인만, `full`이면 전체 소스를 대상으로 한다.
3. 결과를 `schema/findings.schema.json`에 맞는 Finding 배열로만 출력한다.

### 출력
Findings JSON 배열. 이 외 어떤 텍스트도 출력하지 않는다. 렌더링(PR 코멘트/Markdown/SARIF)은 엔진의 렌더러가 담당하며, 이 커맨드는 원시 Findings만 생성한다.

### 실패 처리
- 규칙을 하나도 로드하지 못하면 빈 배열 `[]`을 출력하고 종료한다 (임의 규칙으로 대체하지 않는다).
- `limits.max_diff_lines`를 초과하면 파일 단위 요약 리뷰로 전환한다.
