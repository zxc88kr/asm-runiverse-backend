---
applyTo: "running-service/src/main/java/com/runiverse/running_service/infrastructure/**"
---

# infrastructure 레이어

`port/out`을 구현한다. 기술 단위로 나눈다 — `persistence` · `redis` · `security` · `oauth` · `identifier` 등.

## 네이밍 — 세 접미사를 구분한다

역할이 다르므로 서로 바꿔 쓰지 않는다. 잘못 쓰였으면 지적한다.

| 접미사 | 역할 |
|---|---|
| `*Adapter` | application 포트를 기술 경계에 연결하는 기본 구현체 |
| `*Client` | 외부 제공자와 직접 통신하는 구현 |
| `*Router` | 여러 Client를 선택하면서 application 포트를 구현하는 컴포넌트 |

## 확인할 것

- **도메인 ↔ JPA 변환 담당** — 어댑터가 `toDomain()`으로 도메인 객체를 복원한다.
  JPA 엔티티가 application이나 presentation으로 새어 나가면 지적한다.
- **포트 시그니처 준수** — 구현이 포트 인터페이스에 없는 public 메서드를 늘려서
  application이 그걸 직접 쓰게 되는 구조면 지적한다.
- **어댑터 통합 범위** — 같은 애그리거트와 같은 저장 기술의 포트는 어댑터 하나가
  함께 구현할 수 있다. 이건 지적 대상이 아니다.
- **도메인 오염 금지** — 어댑터의 편의를 위해 도메인 객체에 JPA 애너테이션이나
  기본 생성자를 요구하는 변경이 섞여 있으면 지적한다.

## 지적하지 않을 예외 범위

`docs/architecture.md`가 명시한 현재 리팩터링 예외다. 새 의존을 추가하는 근거로는 쓸 수 없지만,
아래 기존 흐름 자체는 지적 대상이 아니다.

- `SecurityConfig` → `JwtAuthenticationEntryPoint`
- `JwtAuthenticationEntryPoint` → `BlockedTokenValidator` · `ExpiredTokenValidator`
- `BlockedTokenValidator` → `AuthErrorCode`
- `UserPersistenceAdapter.toDomain()`이 `UserOnboard`를 복원하지 않는 현재 흐름

이 범위를 **넓히는** 변경(새로운 security 직접 참조 추가 등)은 지적한다.
