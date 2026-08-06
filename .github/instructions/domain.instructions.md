---
applyTo: "running-service/src/main/java/com/runiverse/running_service/domain/**"
---

# domain 레이어

어떤 레이어도 import하지 않는다. Spring·JPA에도 의존하지 않는다.

## 확인할 것

- **프레임워크 import 금지** — `org.springframework`, `jakarta.persistence`, `lombok` 중
  영속성·DI 관련 애너테이션이 들어오면 지적한다.
- **VO는 생성 시점에 검증** — `record`로 만들어 불변과 값 기준 동등성을 강제하고,
  생성자(compact constructor)에서 검증한다. 식별자·이메일·닉네임처럼 의미가 있는 값을
  원시 타입(`String`, `Long`)으로 그대로 들고 다니면 지적한다.
- **고정된 값 집합은 enum** — `Gender`, `Provider`처럼.
- **애그리거트 루트만 외부에 노출** — `User`가 `OauthUser`·`UserOnboard`를 내부에 든다.
  내부 엔티티가 포트나 외부 레이어에 직접 노출되면 지적한다.
- **의도가 드러나는 메서드로만 상태 변경** — `linkOauth`, `completeOnboarding` 형태.
  무분별한 setter나 외부에서 필드를 직접 조작하는 흐름은 지적한다.
- **생성 방식이 여럿이면 정적 팩토리** — `User.registerWithOauth` 형태.
- **애그리거트 간 참조는 ID로** (Reference by Identity). 애그리거트 내부는 객체 참조를 쓴다.
- **예외는 도메인 예외로** — VO 검증과 애그리거트 불변식 위반은
  `domain/common/exception/BusinessException`을 쓴다.
  `application` 쪽 동명 클래스를 import했으면 지적한다.

## 하위 패키지

`aggregate` · `vo` · `exception`으로 나눈다.

## 주의

도메인 예외는 `GlobalExceptionHandler`에서 전부 500으로 마스킹된다. 사용자에게 400으로
보여줘야 하는 검증이라면 도메인이 아니라 Request DTO의 Bean Validation에 있어야 한다.
도메인 예외로 400을 기대하는 코드가 보이면 지적한다.
