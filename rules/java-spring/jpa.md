# Java/Spring · JPA/영속성 규칙

## JPA-001: 트랜잭션 경계 누락
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: 여러 엔티티에 대한 쓰기 작업(저장/수정/삭제)이 하나의 논리적 단위인데 `@Transactional`로 묶이지 않아 부분 실패 시 데이터 불일치 가능.
- Bad 예시:
  ```java
  public void transfer(Long fromId, Long toId, BigDecimal amount) {
      accountRepository.withdraw(fromId, amount);
      accountRepository.deposit(toId, amount);
  }
  ```
- Good 예시:
  ```java
  @Transactional
  public void transfer(Long fromId, Long toId, BigDecimal amount) {
      accountRepository.withdraw(fromId, amount);
      accountRepository.deposit(toId, amount);
  }
  ```
- 예외: 각 저장소 호출이 이미 자체 트랜잭션으로 원자적이고, 실패해도 재시도로 안전하게 복구 가능함이 명시된 경우.

## JPA-002: 읽기 전용 트랜잭션 미지정
- 심각도: LOW
- 검사 방식: LLM
- 판단 기준: 조회 전용 서비스 메서드에 `@Transactional(readOnly = true)`가 없어 불필요한 dirty checking 비용 발생.
- Bad 예시:
  ```java
  @Transactional
  public OrderResponse get(Long id) { return OrderResponse.from(orderRepository.findById(id).orElseThrow()); }
  ```
- Good 예시:
  ```java
  @Transactional(readOnly = true)
  public OrderResponse get(Long id) { return OrderResponse.from(orderRepository.findById(id).orElseThrow()); }
  ```
- 예외: 조회 중간에 조회수 증가처럼 부수적 쓰기가 실제로 필요한 경우.

## JPA-003: 반복문 내 지연로딩 접근 금지 (N+1)
- 심각도: HIGH
- 검사 방식: LLM (정적분석으로 탐지 불가)
- 판단 기준: 컬렉션 순회 중 연관 엔티티 getter 호출 + fetch join/EntityGraph 부재
- Bad 예시:
  ```java
  orders.forEach(o -> o.getItems().size());
  ```
- Good 예시:
  ```java
  @Query("select o from Order o join fetch o.items")
  ```
- 예외: `@BatchSize` 설정이 명시된 경우

## JPA-004: 벌크 연산 후 영속성 컨텍스트 미갱신
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: `@Modifying` 벌크 UPDATE/DELETE 쿼리 실행 후, 같은 트랜잭션 내에서 영속성 컨텍스트를 비우지 않고(`clearAutomatically`/`EntityManager.clear()`) 이전에 로드된 엔티티를 그대로 사용해 stale 데이터 참조.
- Bad 예시:
  ```java
  @Modifying
  @Query("update Order o set o.status = 'CANCELLED' where o.id in :ids")
  void cancelAll(List<Long> ids);
  // 호출 직후 같은 트랜잭션에서 이미 로드된 order.getStatus() 사용
  ```
- Good 예시:
  ```java
  @Modifying(clearAutomatically = true)
  @Query("update Order o set o.status = 'CANCELLED' where o.id in :ids")
  void cancelAll(List<Long> ids);
  ```
- 예외: 벌크 연산 이후 해당 트랜잭션에서 영향받은 엔티티를 다시 조회하지 않는 경우.

## JPA-005: 편의 메소드 없이 양방향 연관관계 직접 조작
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: 양방향 `@OneToMany`/`@ManyToOne` 관계에서 컬렉션에 `add`만 하고 반대편 FK 필드를 설정하지 않아(또는 그 반대) 연관관계 편의 메소드 없이 한쪽만 갱신. 영속성 컨텍스트 동기화 불일치로 이어짐.
- Bad 예시:
  ```java
  order.getItems().add(item);
  // item.setOrder(order) 누락
  ```
- Good 예시:
  ```java
  public void addItem(OrderItem item) {
      items.add(item);
      item.setOrder(this);
  }
  ```
- 예외: 단방향 연관관계이거나, `mappedBy` 없이 읽기 전용 뷰로만 사용하는 경우.
