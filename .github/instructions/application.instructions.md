---
applyTo: "running-service/src/main/java/com/runiverse/running_service/application/**"
---

# application 레이어

포트를 소유한다. `presentation`은 `port/in`을 호출하고 `infrastructure`가 `port/out`을 구현한다.

## 패키지 구조

```
<domain>/command/     기능별 Command · Handler · Result(선택)
<domain>/port/in/     *Usecase — Handler가 구현
<domain>/port/out/    아웃바운드 인터페이스 · 전용 입출력 모델
<domain>/exception/   외부 확인이 필요한 실패
common/exception/     BusinessException · ErrorCode (유스케이스용)
```

## 확인할 것

- **infrastructure 직접 참조 금지** — 구체 어댑터·JPA 엔티티·Redis 클라이언트를 직접
  import하면 지적한다. 반드시 `port/out` 인터페이스를 거친다.
- **Handler는 `*Usecase`를 구현** — port/in 인터페이스 없이 컨트롤러가 Handler를 직접
  호출하는 구조면 지적한다.
- **`Result`는 반환값이 있을 때만** 둔다.
- **포트는 작고 응집되게** — 사용 유스케이스나 변경 이유가 다르면 포트를 분리한다.
  Handler에는 실제로 쓰는 포트만 주입한다. 안 쓰는 포트가 주입돼 있으면 지적한다.
- **트랜잭션 경계는 application** — 보통 Handler에 `@Transactional`을 둔다. 다만 Redis
  전용처럼 DB를 쓰지 않는 유스케이스는 경계가 없는 것이 정상이니 지적하지 않는다.
- **예외 레이어 확인** — 외부를 확인해야 아는 실패(중복·조회·인증·연동)가 application이다.
  `domain` 쪽 동명 `BusinessException`·`ErrorCode`를 import했으면 지적한다.

## ErrorCode 추가 시

`ErrorCode` enum에 상수를 추가했다면 다음 둘을 함께 확인한다.

1. `presentation/common/exception/GlobalExceptionHandler.toStatus()` — 누락 시 컴파일 에러
2. `presentation/common/exception/ErrorExposurePolicy.EXPOSED_CODES` — **누락 시 런타임 500 마스킹**

2번이 이 프로젝트에서 가장 놓치기 쉬운 지점이다. 새 코드가 400이 아닌 상태로 매핑되는데
`EXPOSED_CODES`에 없다면, 의도적 비노출인지 누락인지 PR에서 확인을 요청한다.

## 참고 기준

구현 스타일은 `application/auth/command/signup`·`login` 패키지가 기준이다.
