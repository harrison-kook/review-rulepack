# Java/Spring · 서비스 계층 테스트케이스

## TC-SVC-001: 트랜잭션 중간 실패 시 전체 롤백
- 대상 계층: Service
- 우선순위: P1
- 탐색 힌트: `@Transactional` 메서드 내 다중 저장소 호출
- Given: 두 번째 저장소 호출에서 예외 발생하도록 설정
- When: 트랜잭션 메서드 실행
- Then: 첫 번째 저장소 호출 결과도 롤백되어 DB에 반영되지 않음

## TC-SVC-002: 낙관적 락 충돌 시 재시도 또는 명시적 예외
- 대상 계층: Service
- 우선순위: P2
- 탐색 힌트: `@Version` 필드가 있는 엔티티를 수정하는 서비스 메서드
- Given: 동일 엔티티를 두 트랜잭션이 동시에 수정
- When: 두 번째 커밋 시도
- Then: `OptimisticLockException` 또는 팀 정의 커스텀 예외 발생, 데이터 불일치 없음

## TC-SVC-003: 도메인 유효성 검증 실패 시 저장소 호출 없음
- 대상 계층: Service
- 우선순위: P1
- 탐색 힌트: `validate`, `assert`로 시작하는 도메인 검증 메서드
- Given: 도메인 규칙을 위반하는 입력(예: 음수 수량)
- When: 서비스 메서드 호출
- Then: 예외 발생, Repository의 `save`/`update`가 호출되지 않음 (Mockito `verify(times(0))`)

## TC-SVC-004: 외부 API 실패 시 서비스 예외로 변환
- 대상 계층: Service
- 우선순위: P2
- 탐색 힌트: `RestTemplate`, `WebClient`, `FeignClient` 호출부
- Given: 외부 API가 타임아웃/5xx 응답
- When: 서비스 메서드 호출
- Then: 원본 예외가 아닌 도메인 예외로 변환되어 전파, 원인(cause) 보존
