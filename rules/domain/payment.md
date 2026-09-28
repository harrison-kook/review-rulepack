# 도메인 · 결제 규칙

## PAY-001: 금액 필드에 float/double 사용
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 금액, 수수료, 환율 등 정밀도가 중요한 필드/파라미터/DB 컬럼 매핑 타입이 `float`/`double`. 부동소수점 오차로 인한 금액 불일치 위험.
- Bad 예시:
  ```java
  private double amount;
  ```
- Good 예시:
  ```java
  private BigDecimal amount;
  ```
- 예외: 화면 표시용 비율(%) 등 금전 계산에 직접 관여하지 않는 값.

## PAY-002: 결제 승인 멱등성 미보장
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 결제 승인/취소 API가 동일 요청(주문번호, idempotency key 등)의 재호출을 막는 장치(유니크 제약, 상태 체크, 멱등키 저장) 없이 매번 PG 호출 및 레코드 생성을 수행.
- Bad 예시:
  ```java
  public void approve(String orderId) {
      pgClient.approve(orderId);
      paymentRepository.save(new Payment(orderId));
  }
  ```
- Good 예시:
  ```java
  public void approve(String orderId) {
      if (paymentRepository.existsByOrderIdAndStatus(orderId, APPROVED)) return;
      pgClient.approve(orderId);
      paymentRepository.save(new Payment(orderId));
  }
  ```
- 예외: 멱등성이 PG사 API 레벨(멱등키 헤더)에서 이미 보장되고 그 사실이 코드/주석으로 확인 가능한 경우.

## PAY-003: 재시도 정책 없이 외부 PG 호출
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: 네트워크 오류/타임아웃이 발생할 수 있는 PG(결제대행사) API 호출에 재시도, 타임아웃 설정, 서킷브레이커 등 장애 대응 정책이 전혀 없음.
- Bad 예시:
  ```java
  ResponseEntity<PgResponse> response = restTemplate.postForEntity(pgUrl, request, PgResponse.class);
  ```
- Good 예시:
  ```java
  @Retryable(maxAttempts = 3, backoff = @Backoff(delay = 500))
  ResponseEntity<PgResponse> callPg(PgRequest request) { ... }
  ```
- 예외: 승인 API처럼 재시도 시 중복 결제 위험이 더 큰 경우, 대신 멱등키(PAY-002)로 처리했다면 인정.

## PAY-004: 부분 취소 시 금액 검증 누락
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 부분 취소 요청 금액이 원 결제 금액과 기취소 금액 합계를 초과하는지 검증하지 않음. 초과 취소로 인한 금액 정합성 오류 가능.
- Bad 예시:
  ```java
  public void partialCancel(String orderId, BigDecimal cancelAmount) {
      pgClient.cancel(orderId, cancelAmount);
  }
  ```
- Good 예시:
  ```java
  public void partialCancel(String orderId, BigDecimal cancelAmount) {
      Payment payment = paymentRepository.findByOrderId(orderId).orElseThrow();
      BigDecimal remaining = payment.getAmount().subtract(payment.getCancelledAmount());
      if (cancelAmount.compareTo(remaining) > 0) throw new ExceedsCancellableAmountException();
      pgClient.cancel(orderId, cancelAmount);
  }
  ```
- 예외: 없음.

## PAY-005: 상태 전이 검증 없이 결제 상태 직접 변경
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 결제 상태(`READY`/`APPROVED`/`CANCELLED`/`FAILED` 등)를 전이 규칙 검증 없이 setter로 직접 변경. 이미 취소된 결제를 다시 승인 처리하는 등 잘못된 전이 허용.
- Bad 예시:
  ```java
  payment.setStatus(APPROVED);
  ```
- Good 예시:
  ```java
  payment.approve(); // 엔티티 내부에서 현재 상태가 READY인지 검증 후 전이
  ```
- 예외: 관리자 강제 보정(reconciliation) 도구처럼 별도 권한 체크와 감사 로그가 있는 경우.
