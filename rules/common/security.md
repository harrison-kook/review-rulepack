# 공통 · 보안 규칙

## SEC-001: SQL Injection
- 심각도: HIGH
- 검사 방식: LLM (정적분석 보조 가능, 최종 판단은 LLM)
- 판단 기준: 사용자 입력값을 문자열 결합/포맷팅으로 SQL·JPQL·네이티브 쿼리에 직접 삽입. PreparedStatement 바인딩 파라미터, `@Param`, JPA 파생 쿼리 메서드가 아닌 경우.
- Bad 예시:
  ```java
  String sql = "SELECT * FROM users WHERE name = '" + name + "'";
  jdbcTemplate.queryForList(sql);
  ```
- Good 예시:
  ```java
  jdbcTemplate.queryForList("SELECT * FROM users WHERE name = ?", name);
  ```
- 예외: 상수/시스템 생성 값만 결합하고 사용자 입력이 전혀 섞이지 않는 경우.

## SEC-002: 시크릿 하드코딩
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: API 키, 비밀번호, 토큰, DB 접속정보 등이 소스코드에 리터럴로 존재. `application.yml`이라도 평문으로 커밋된 프로덕션 시크릿 포함.
- Bad 예시:
  ```java
  String apiKey = "sk-live-9f8c2e...";
  ```
- Good 예시:
  ```java
  @Value("${payment.api-key}")
  private String apiKey;
  ```
- 예외: 테스트 코드의 가짜/샘플 키(`test-`, `dummy-` 접두어 등 명백히 목적이 드러나는 경우)는 MEDIUM으로 하향.

## SEC-003: 민감정보 로깅
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 비밀번호, 카드번호, 주민등록번호, 토큰, 세션ID 등이 마스킹 없이 `log.info/debug/error` 등으로 출력.
- Bad 예시:
  ```java
  log.info("login attempt: {} / {}", username, password);
  ```
- Good 예시:
  ```java
  log.info("login attempt: {}", username);
  ```
- 예외: 로그 레벨이 TRACE이고 로컬 개발 프로파일 전용으로 명시적으로 분기된 경우 MEDIUM.

## SEC-004: 안전하지 않은 역직렬화
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 신뢰할 수 없는 입력을 `ObjectInputStream`, `XMLDecoder` 등 다형성 역직렬화 API로 직접 처리하거나, Jackson에서 `enableDefaultTyping` 등으로 타입 화이트리스트 없이 임의 타입 역직렬화를 허용.
- Bad 예시:
  ```java
  ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
  Object obj = ois.readObject();
  ```
- Good 예시:
  ```java
  MyDto dto = objectMapper.readValue(request.getInputStream(), MyDto.class);
  ```
- 예외: 내부 배치 프로세스 간에만 사용되고 외부 입력이 절대 섞이지 않는 파일.

## SEC-005: 인증/인가 누락
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 상태 변경(POST/PUT/PATCH/DELETE) 또는 민감 데이터 조회 엔드포인트에 `@PreAuthorize`, 시큐리티 필터, 세션/토큰 검증 등 인가 체크가 전혀 없음.
- Bad 예시:
  ```java
  @DeleteMapping("/users/{id}")
  public void deleteUser(@PathVariable Long id) { ... }
  ```
- Good 예시:
  ```java
  @PreAuthorize("hasRole('ADMIN')")
  @DeleteMapping("/users/{id}")
  public void deleteUser(@PathVariable Long id) { ... }
  ```
- 예외: 헬스체크, 공개 조회용으로 설계 문서/PR 설명에 명시된 엔드포인트.
