# 팀 고유 규칙 (템플릿)

이 파일은 팀별 컨벤션을 넣는 자리다. `TEAM-` 접두어를 사용하고, 형식은 다른 `rules/*.md`와 동일하게 유지한다.
아래는 작성 예시이며, 실제 파일럿 팀 컨벤션이 정해지면 교체한다.

## TEAM-001: (예시) 커밋되는 컨트롤러 응답은 항상 ApiResponse<T>로 감싼다
- 심각도: LOW
- 검사 방식: LLM
- 판단 기준: `@RestController` 메서드의 반환 타입이 팀 공통 래퍼(`ApiResponse<T>`) 없이 도메인 객체/DTO를 그대로 반환.
- Bad 예시:
  ```java
  @GetMapping("/orders/{id}")
  OrderResponse get(@PathVariable Long id) { ... }
  ```
- Good 예시:
  ```java
  @GetMapping("/orders/{id}")
  ApiResponse<OrderResponse> get(@PathVariable Long id) { ... }
  ```
- 예외: 파일 다운로드, 리다이렉트 등 래핑이 의미 없는 응답.

---

> 새 규칙 추가 시 `TEAM-XXX` 순번을 이어서 사용하고, 팀 리뷰(승인)를 거쳐 병합한다. 접근 권한 정책은 설계서 13장 미결 사항 참고.
