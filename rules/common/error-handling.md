# 공통 · 예외 처리 규칙

## ERR-001: 예외 무시(swallow)
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: `catch` 블록이 비어 있거나 로그 한 줄 없이 무시. 장애 원인 추적이 불가능해지는 경우.
- Bad 예시:
  ```java
  try {
      process();
  } catch (Exception e) {
      // ignore
  }
  ```
- Good 예시:
  ```java
  try {
      process();
  } catch (IOException e) {
      log.error("process failed", e);
      throw new ProcessException(e);
  }
  ```
- 예외: 의도적으로 실패를 무시해도 되는 로직이고, 그 이유가 주석으로 명시된 경우 INFO로 하향.

## ERR-002: 광범위한 Exception catch
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: 구체적 예외 타입 대신 `catch (Exception e)` 또는 `catch (Throwable t)`로 뭉뚱그려 처리하여 의도하지 않은 예외(예: `NullPointerException`, `OutOfMemoryError`)까지 삼킴.
- Bad 예시:
  ```java
  try {
      payment.approve();
  } catch (Exception e) {
      return ResponseEntity.ok().build();
  }
  ```
- Good 예시:
  ```java
  try {
      payment.approve();
  } catch (PaymentGatewayException e) {
      return ResponseEntity.status(502).build();
  }
  ```
- 예외: 최상위 컨트롤러/필터의 글로벌 예외 핸들러처럼 마지막 방어선 역할이 명확한 경우.

## ERR-003: 예외 메시지에 민감정보 포함
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 예외 메시지나 클라이언트 응답에 스택트레이스, 내부 쿼리, 시크릿, 개인정보가 그대로 노출.
- Bad 예시:
  ```java
  catch (SQLException e) {
      return ResponseEntity.status(500).body(e.getMessage());
  }
  ```
- Good 예시:
  ```java
  catch (SQLException e) {
      log.error("query failed", e);
      return ResponseEntity.status(500).body("internal error");
  }
  ```
- 예외: 없음 (내부 로그로 남기는 것은 무관하나, 외부 응답 노출은 항상 위반).

## ERR-004: 체크 예외 변환 시 원인(cause) 유실
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: 체크 예외를 런타임 예외로 감쌀 때 원본 예외를 `cause`로 전달하지 않아 원인 추적이 끊김.
- Bad 예시:
  ```java
  catch (IOException e) {
      throw new RuntimeException("failed");
  }
  ```
- Good 예시:
  ```java
  catch (IOException e) {
      throw new RuntimeException("failed", e);
  }
  ```
- 예외: 원본 예외에 시크릿/민감정보가 포함되어 의도적으로 제거하는 경우, 대신 로그에 원인을 남겼다면 인정.

## ERR-005: 리소스 미해제
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: `InputStream`, `Connection`, `Statement` 등 `Closeable` 자원을 try-with-resources나 `finally` 없이 열기만 하고 닫지 않음.
- Bad 예시:
  ```java
  FileInputStream fis = new FileInputStream(file);
  byte[] data = fis.readAllBytes();
  ```
- Good 예시:
  ```java
  try (FileInputStream fis = new FileInputStream(file)) {
      byte[] data = fis.readAllBytes();
  }
  ```
- 예외: Spring이 관리하는 빈(예: `DataSource`가 관리하는 `Connection`)처럼 프레임워크가 생명주기를 책임지는 경우.
