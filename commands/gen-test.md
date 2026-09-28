---
description: "[2단계 스텁] testcases/*.md 기반으로 테스트를 생성·실행"
---

## /gen-test

> 상태: 로드맵 2단계 작업 시 구체화 예정. `agents/tester.md`가 구체화되는 시점에 이 커맨드도 함께 완성한다.

### 예정 동작
`tester` 서브에이전트(`agents/tester.md`)를 호출해 `testcases/**/*.md`에 정의된 시나리오 중 변경된 클래스와 관련된 것을 찾아 테스트를 생성·실행하고, 커버리지/뮤테이션 결과를 함께 리포트한다.
