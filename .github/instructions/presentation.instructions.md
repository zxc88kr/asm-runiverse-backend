---
applyTo: "running-service/src/main/java/com/runiverse/running_service/presentation/**"
---

# presentation 레이어

컨트롤러 + DTO. `port/in`만 호출한다.

## 확인할 것

- **`port/in`만 호출** — Handler 구현체나 `port/out`, infrastructure를 직접 참조하면 지적한다.
- **경계 변환은 컨트롤러가** — `SignUpRequest` → `SignUpCommand` 변환은 컨트롤러 책임이다.
  Request DTO를 application 레이어로 그대로 넘기면 지적한다.
- **400 검증은 Bean Validation으로** — 도메인 예외는 전부 500으로 마스킹되므로,
  사용자에게 400으로 보여줘야 하는 검증은 Request DTO의 `@NotNull`·`@Size` 등이 만든다.
- **예외 변환 위치** — 컨트롤러 처리 중 예외는 `GlobalExceptionHandler`,
  인증 진입 실패는 `AuthenticationEntryPoint`에서 변환한다.

## API 표면 규칙

- **Base path**: `/api/v1`
- **필드명**: JSON은 camelCase (DB 컬럼 snake_case는 백엔드에서 매핑)
- **ID 타입**: `userId`만 UUID, 그 외 ID는 Long. 새 DTO에서 이 규칙을 어기면 지적한다.
- **날짜/시간**: ISO 8601, UTC (`2026-07-20T04:00:00Z`)
- **enum**: DB·API 동일한 영문 코드, 변환 매핑 없음
- **에러 응답**: `{ code, message }` 평면 구조 — `error` 래핑이나 status 필드를 추가하면 지적한다
- **페이지네이션**: 커서 기반 `?cursor=&limit=`, 응답은 `{ items, nextCursor }`
- **토글형 액션**: POST(등록)/DELETE(취소)로 분리. 갱신된 상태·카운트가 필요하면 `200 OK`,
  반환할 값이 없으면 `204 No Content`

## 물리량 단위 접미사

단위는 풀네임으로 명시한다. 통용어 `Spm`(케이던스)만 약어를 허용한다.

- `...Meters` / `...Seconds` / `...SecondsPerKm` / `...MetersPerSecond` / `...Degrees` / `...Kg` / `...Cm`
- 거리는 전부 미터, 페이스는 초/km 정수
- 예: `totalDistanceMeters`, `averagePaceSecondsPerKm`, `weightKg`

단위 없는 필드명(`distance`, `pace`, `weight`)이 새로 추가되면 지적한다.

## 인증

- Bearer 토큰, Access+Refresh 이원화
- Refresh 시 rotation — access·refresh 모두 재발급, 이전 refreshToken 무효화
- 로그아웃 시 access 토큰 블랙리스트 처리 (`401 TOKEN_BLOCKED`)

## 에러 코드 등록

`ErrorCode`가 추가된 PR에서는 `ErrorExposurePolicy.EXPOSED_CODES` 등록 여부를 확인한다.
누락되면 컴파일·테스트를 통과하고도 런타임에 500으로 마스킹된다.

`USER_NOT_FOUND`, `INVALID_EMAIL_CREDENTIALS`, `INVALID_PASSWORD_CREDENTIALS`의 비노출은
계정 존재 여부를 감추기 위한 의도이므로 지적하지 않는다.
