# Runiverse 백엔드 리뷰 지침

원격 동반 러닝 플랫폼의 API 서버. Java + Spring Boot, Gradle 멀티 모듈(`running-service`).
클린 아키텍처 + DDD를 따르며 패키지 루트는 `com.runiverse.running_service`.

리뷰 코멘트는 한국어로 작성한다.

## 아키텍처 요약

```
presentation ──▶ [port/in] application [port/out] ◀── infrastructure
                                │
                                ▼
                              domain
```

의존 방향은 항상 바깥에서 안쪽(application/domain)으로 향한다. 레이어별 세부 규칙은
`.github/instructions/` 아래 경로별 지침을 따른다.

상세 규칙 문서는 `docs/architecture.md`, `docs/code-convention.md`, `docs/api-convention.md`에 있다.

## 우선순위 높은 검토 항목

1. **레이어 의존 방향 위반** — 특히 domain이 Spring·JPA를 import하는 경우
2. **에러 코드 등록 누락** — 아래 별도 항목 참고
3. **API 표면 규칙 위반** — 필드 단위 접미사, 에러 응답 형태, ID 타입
4. **애그리거트 불변식이 도메인 밖에서 깨지는 경우**

## 에러 코드 등록 규칙

application 에러 코드를 추가할 때 손대야 하는 곳이 셋이다.

| 위치 | 누락 시 |
|---|---|
| `application/common/exception/ErrorCode` | — |
| `GlobalExceptionHandler.toStatus()` | 컴파일 에러 (exhaustive switch) |
| `ErrorExposurePolicy.EXPOSED_CODES` | **런타임에 조용히 500으로 마스킹** |

`toStatus()`는 컴파일러가 잡아주지만 `EXPOSED_CODES`는 못 잡는다. 새 `ErrorCode`가
추가된 PR에서는 **`EXPOSED_CODES` 등록 여부를 반드시 확인**한다. 컴파일과 테스트를
모두 통과하고도 클라이언트에는 500이 나가는 유일한 지점이다.

단, 비노출이 의도인 코드가 이미 있다. 아래는 지적하지 않는다.

- `USER_NOT_FOUND` — 계정 존재 여부를 노출하지 않기 위해 의도적으로 마스킹
- `INVALID_EMAIL_CREDENTIALS`, `INVALID_PASSWORD_CREDENTIALS` — 같은 이유

새로 추가된 코드가 이 부류라면 비노출 근거가 코드나 테스트에 남아 있는지 확인한다.

## 지적하지 않을 것

- `running_service` 패키지명의 언더스코어 — 베이스 패키지라 리네임하지 않는다
- `docs/architecture.md` "구현 스타일 기준"에 명시된 리팩터링 예외 범위
- 포맷팅 — 루트 `.editorconfig`와 Google Java Style이 담당한다
