# Step 6 — AOP: 동적 프록시

## 학습 목표

- JDK 동적 프록시의 동작 원리를 이해한다.
- "프록시 기반 AOP"의 능력과 한계(자기 호출 문제, final/private 한계)를 구현으로 체득한다.
- `BeanPostProcessor`가 왜 AOP의 접점인지 이해한다.

## 실무 미스터리

- **왜 `@Transactional`을 같은 클래스 내부 호출(self-invocation)에서 쓰면 안 먹을까?** → 이 단계에서 내 프록시로 직접 재현한다. 실무자에게 "아하"가 가장 큰 구간.
- 스프링 빈을 주입받으면 가끔 클래스 이름에 `$Proxy` / `$$SpringCGLIB`이 붙어 있는 이유는?
- **Kotlin 프로젝트에는 왜 `kotlin("plugin.spring")`(all-open)이 거의 필수로 붙어 있을까?** → Kotlin 클래스는 기본 `final`이다. 이 단계에서 그 벽에 직접 부딪힌다.

## 구현할 것

- [ ] `InvocationHandler` 기반 프록시 팩토리
- [ ] `@Around` 스타일 어드바이스 인터페이스 (`Object around(Invocation inv)` — `inv.proceed()`로 원본 호출)
- [ ] 간단 포인트컷: **애노테이션 기반 매칭** ("이 애노테이션이 붙은 메서드만 어드바이스 적용")
- [ ] **BeanPostProcessor 흉내**: Step 4 컨테이너에 "빈 생성 직후 후처리 훅"을 추가하고, 포인트컷에 매칭되는 빈을 프록시로 감싸 **컨테이너에 프록시를 대신 등록**
- [ ] 적용 예제 (boardy 서비스 계층):
  - [ ] `@Timed` — 실행 시간 로깅
  - [ ] `@Retry` — 예외 시 재시도
- [ ] **자기 호출(self-invocation) 문제 재현 테스트** — 같은 클래스 안에서 `this.method()` 호출 시 어드바이스가 적용되지 않음을 증명
- [ ] (문서로만 정리) CGLIB/ByteBuddy 클래스 프록시와의 차이, 인터페이스 없는 빈은 어떻게 프록시하나
- [ ] (문서로만 정리) **Kotlin `final` 클래스 × 상속 기반 프록시**: 인터페이스 없는 Kotlin 클래스를 CGLIB 방식으로 프록시하려면 왜 실패하나 → kotlin-spring(all-open) 플러그인이 하는 일 정리

## Spring 대응표

| 내가 만드는 것 | Spring의 대응물 |
|---|---|
| 프록시 팩토리 | `java.lang.reflect.Proxy` + `ProxyFactory` |
| `InvocationHandler` | `JdkDynamicAopProxy` (Spring 내부) |
| 어드바이스 인터페이스 | `@Aspect` / `@Around`, `MethodInterceptor` |
| 애노테이션 포인트컷 | `@annotation(...)` 포인트컷 표현식 |
| 컨테이너 후처리 훅 | `BeanPostProcessor.postProcessAfterInitialization` |
| 자동 프록시 등록 | `AnnotationAwareAspectJAutoProxyCreator` |
| (문서) 클래스 프록시 | CGLIB, `proxyTargetClass=true` |
| (문서) Kotlin `final` 클래스의 벽 | kotlin-spring(all-open) 플러그인 |

**비교 포인트**: "빈을 꺼냈더니 원본이 아니라 프록시더라"가 어떻게 가능한지 — 컨테이너가 빈 생성 직후 후처리 훅에서 객체를 **교체**하기 때문. 이 훅 하나로 AOP, `@Transactional`, `@Async` 등이 모두 설명된다.

## 완료 기준 (DoD)

- [ ] `@Timed` 붙은 서비스 메서드 호출 시 실행 시간 로그 출력, 안 붙은 메서드는 미출력
- [ ] `@Retry` 붙은 메서드가 예외 시 지정 횟수만큼 재시도
- [ ] **자기 호출 시 어드바이스 미적용을 테스트로 증명** (프록시 경유 호출 vs `this` 호출 비교)
- [ ] 프록시 적용이 컨테이너 후처리 훅에서 자동으로 일어남 — `app` 모듈에 수동 래핑 코드 0줄

## 심화 과제 (Optional)

- 어드바이스 체이닝: 한 메서드에 `@Timed` + `@Retry` 동시 적용 시 프록시 중첩(또는 어드바이스 체인) 순서 제어
- 어드바이스 적용 순서 지정 (`@Order` 흉내)
- ByteBuddy로 클래스 프록시를 만들어 인터페이스 없는 빈도 프록시해보기

## 권장 학습 자료 키워드

- JDK Dynamic Proxy (`Proxy.newProxyInstance`)
- 토비의 스프링 Vol.1 — 6장 (AOP)
- Spring 공식 문서 "Proxying Mechanisms"
- CGLIB vs JDK proxy

## 학습 노트

(진행하며 기록)

## 회고: Spring이라면?

(완료 후 작성)
