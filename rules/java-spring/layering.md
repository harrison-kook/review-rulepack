# Java/Spring · 계층 분리 규칙

## LAY-001: Controller에서 Repository 직접 접근
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: `@Controller`/`@RestController` 클래스가 `@Repository` 또는 `*Repository` 타입을 직접 주입받아 사용. Service 계층을 우회.
- Bad 예시:
  ```java
  @RestController
  class OrderController {
      private final OrderRepository orderRepository;
      @GetMapping("/orders/{id}")
      Order get(@PathVariable Long id) { return orderRepository.findById(id).orElseThrow(); }
  }
  ```
- Good 예시:
  ```java
  @RestController
  class OrderController {
      private final OrderService orderService;
      @GetMapping("/orders/{id}")
      OrderResponse get(@PathVariable Long id) { return orderService.get(id); }
  }
  ```
- 예외: 단순 조회 전용 CQRS 조회 모델을 팀 컨벤션으로 명시적으로 허용한 경우.

## LAY-002: Service가 HTTP 계층 객체에 의존
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: `@Service` 클래스 메서드 시그니처나 필드에 `HttpServletRequest`, `HttpServletResponse`, `HttpSession` 등이 직접 등장. 비즈니스 로직이 전송 계층에 결합됨.
- Bad 예시:
  ```java
  @Service
  class OrderService {
      void create(HttpServletRequest request) { String token = request.getHeader("X-Token"); ... }
  }
  ```
- Good 예시:
  ```java
  @Service
  class OrderService {
      void create(String token, OrderCommand command) { ... }
  }
  ```
- 예외: 파일 업로드 등 Servlet API의 `MultipartFile`처럼 Spring이 추상화를 제공하는 타입은 위반 아님.

## LAY-003: Entity를 API 응답으로 직접 노출
- 심각도: HIGH
- 검사 방식: LLM
- 판단 기준: `@Entity` 클래스(또는 JPA 프록시)가 컨트롤러 메서드의 반환 타입이나 `@RequestBody`로 그대로 사용됨. DTO 변환 계층 부재.
- Bad 예시:
  ```java
  @GetMapping("/users/{id}")
  User get(@PathVariable Long id) { return userRepository.findById(id).orElseThrow(); }
  ```
- Good 예시:
  ```java
  @GetMapping("/users/{id}")
  UserResponse get(@PathVariable Long id) { return UserResponse.from(userService.get(id)); }
  ```
- 예외: 내부 전용 관리자 도구로 외부에 노출되지 않고 PR 설명에 명시된 경우 MEDIUM.

## LAY-004: Controller에 비즈니스 로직 위치
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: 컨트롤러 메서드 내부에 분기/계산/상태 변경 등 도메인 로직이 여러 줄 이상 존재. 단순 위임(delegation)을 넘어섬.
- Bad 예시:
  ```java
  @PostMapping("/orders/{id}/cancel")
  void cancel(@PathVariable Long id) {
      Order order = orderRepository.findById(id).orElseThrow();
      if (order.getStatus() == PAID) { order.setStatus(CANCELLED); refund(order); }
      orderRepository.save(order);
  }
  ```
- Good 예시:
  ```java
  @PostMapping("/orders/{id}/cancel")
  void cancel(@PathVariable Long id) { orderService.cancel(id); }
  ```
- 예외: 입력 검증(널 체크, 형식 검증) 수준의 짧은 로직은 위반 아님.

## LAY-005: Service 간 순환 의존
- 심각도: MEDIUM
- 검사 방식: LLM
- 판단 기준: 두 개 이상의 `@Service`가 서로를 직접 주입받아 순환 참조를 형성 (`A → B → A`). `@Lazy`로 우회했더라도 설계상 순환은 위반.
- Bad 예시:
  ```java
  @Service class OrderService { private final PaymentService paymentService; }
  @Service class PaymentService { private final OrderService orderService; }
  ```
- Good 예시:
  ```java
  @Service class OrderService { private final PaymentService paymentService; }
  @Service class PaymentService { /* OrderService를 참조하지 않고 이벤트/콜백으로 분리 */ }
  ```
- 예외: 없음 (순환 의존은 항상 재설계 대상).
