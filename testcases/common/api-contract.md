# 공통 · API 계약 테스트케이스

## TC-API-001: 필수 요청 필드 누락 시 400 응답
- 대상 계층: Controller
- 우선순위: P1
- 탐색 힌트: `@RestController`, `@Valid`/`@Validated` 사용 엔드포인트
- Given: 요청 DTO의 `@NotNull`/`@NotBlank` 필드가 비어 있음
- When: 해당 엔드포인트로 요청 전송
- Then: HTTP 400 응답, 유효성 오류 필드명이 응답 바디에 포함

## TC-API-002: 존재하지 않는 리소스 조회 시 404 응답
- 대상 계층: Controller
- 우선순위: P1
- 탐색 힌트: `findById`, `getOrThrow`, `*NotFoundException` 처리 경로
- Given: 존재하지 않는 ID
- When: 단건 조회 엔드포인트 호출
- Then: HTTP 404 응답, 500이 아님

## TC-API-003: 인가되지 않은 사용자의 보호된 엔드포인트 접근 시 403 응답
- 대상 계층: Controller / Security
- 우선순위: P1
- 탐색 힌트: `@PreAuthorize`, `SecurityFilterChain`, 역할(Role) 체크 로직
- Given: 필요한 역할이 없는 인증된 사용자
- When: 보호된 엔드포인트 호출
- Then: HTTP 403 응답, 리소스 데이터가 응답에 노출되지 않음

## TC-API-004: 페이지네이션 파라미터 경계값 처리
- 대상 계층: Controller
- 우선순위: P2
- 탐색 힌트: `Pageable`, `page`/`size` 쿼리 파라미터
- Given: `size=0` 또는 음수 `page` 값
- When: 목록 조회 엔드포인트 호출
- Then: 400 응답 또는 안전한 기본값으로 처리, 예외로 500 발생하지 않음
