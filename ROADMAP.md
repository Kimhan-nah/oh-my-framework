# Java 웹 서버 내부 원리 학습 로드맵

Spring을 "쓸 줄 아는" 상태에서 "왜 그렇게 생겼는지 아는" 상태로 가기 위한 학습 프로젝트.

## 1. 프로젝트 목표와 방식

- **혼합 방식**: 전반부(Step 1~7)는 프레임워크 없이 순수 Java로 미니 스프링(`web-core`)을 직접 만들고, 후반부(Step 8)에 같은 애플리케이션을 실제 Spring Boot로 포팅해 내가 만든 것과 비교한다.
- **Java 21 LTS**: record, 가상 스레드(JEP 444), 패턴 매칭 등 모던 문법을 적극 활용한다.
- **기간 없음, DoD 기반**: 각 단계는 마감일 대신 검증 가능한 **완료 기준(Definition of Done)** 을 가진다. DoD를 만족하면 `git tag step-N`을 찍고 다음 단계로 넘어간다.
- **최종 결과물**: "내가 만든 미니 스프링과 실제 Spring Boot가 동일한 통합 테스트를 통과한다" — 이것이 이 프로젝트의 끝 그림이다.

## 2. 관통 예제: boardy (미니 게시판)

모든 단계에서 **같은 애플리케이션을 점점 프레임워크 위로 올려 리팩터링**한다. 도메인이 단순해야 프레임워크 학습에 집중할 수 있다.

### API 스펙 (최종)

| 메서드 | 경로 | 설명 | 인증 |
|---|---|---|---|
| POST | `/users` | 회원가입 | 공개 |
| POST | `/login` | 로그인 (세션 쿠키 발급) | 공개 |
| GET | `/posts` | 게시글 목록 | 공개 |
| GET | `/posts/{id}` | 게시글 단건 조회 | 공개 |
| POST | `/posts` | 게시글 작성 (**작성자 포인트 +10 적립**) | 필요 |
| PUT | `/posts/{id}` | 게시글 수정 | 필요 |
| DELETE | `/posts/{id}` | 게시글 삭제 | 필요 |
| POST | `/posts/{id}/comments` | 댓글 작성 | 필요 |

시나리오가 학습 장치를 겸한다:
- **글 작성 시 포인트 적립** → 두 테이블 쓰기 = Step 7 트랜잭션 롤백 검증의 실전 시나리오
- **쓰기 API 인증** → Step 5 Filter/Interceptor의 실전 시나리오

### 단계별 진화

| 단계 | boardy의 모습 |
|---|---|
| Step 2 | in-memory Map 저장소, 핸들러 람다로 CRUD |
| Step 3 | `@Controller` 클래스로 리팩터링 (인스턴스는 수동 `new`) |
| Step 4 | 전 계층 빈 전환, 수동 `new` 제거 |
| Step 5 | 세션 로그인 + `@LoginRequired` 인증 |
| Step 6 | 서비스 계층에 `@Timed`/`@Retry` AOP 적용 |
| Step 7 | 저장소를 H2 + MyJdbcTemplate로 교체, `@MyTransactional` 적용 |
| Step 8 | 동일 API를 Spring Boot로 재작성 (`app-spring`) |

## 3. 전체 단계 지도

| Step | 제목 | 한 줄 요약 |
|---|---|---|
| 0 | [프로젝트 셋업](docs/step-00-setup.md) | Gradle 멀티모듈 + Java 21 toolchain 골격 |
| 1 | [소켓 HTTP 서버](docs/step-01-http-server.md) | HTTP는 그냥 텍스트다 — 파서, keep-alive, 스레드 모델 3종 |
| 2 | [서블릿 추상화 + 라우팅](docs/step-02-servlet-routing.md) | Handler 계약과 path variable 라우터 |
| 3 | [애노테이션 MVC](docs/step-03-annotation-mvc.md) | DispatcherServlet = 리플렉션 라우팅 + 바인딩 + 변환 |
| 4 | [DI 컨테이너](docs/step-04-di-container.md) | 컴포넌트 스캔, 생성자 주입, 싱글턴, 순환 참조 |
| 5 | [Filter / Interceptor](docs/step-05-filter-interceptor.md) | 체인 패턴과 세션 인증, 필터 vs 인터셉터의 구조적 차이 |
| 6 | [AOP 동적 프록시](docs/step-06-aop-proxy.md) | JDK 프록시 + BeanPostProcessor 훅 = 자동 프록시 |
| 7 | [JDBC 추상화 + 트랜잭션](docs/step-07-jdbc-transaction.md) | JdbcTemplate과 ThreadLocal 커넥션 동기화, @Transactional의 정체 |
| 8 | [Spring Boot 포팅 & 비교](docs/step-08-spring-porting.md) | 진짜 스프링을 디버거로 추적하며 최종 대응표 완성 |

### 의존 순서

```
Step 1 (소켓 HTTP) → Step 2 (서블릿 추상화+라우팅) → Step 3 (MVC, 수동 등록)
                                                        ↓
                              Step 4 (DI, MVC와 통합) ←─┘
                                    ↓
                              Step 5 (Filter/Interceptor + 인증)
                                    ↓
                              Step 6 (AOP 프록시)          ← DI의 후처리 훅에 의존
                                    ↓
                              Step 7 (JDBC + 선언적 트랜잭션) ← AOP 프록시에 의존
                                    ↓
                              Step 8 (Spring Boot 포팅)
```

- DI(Step 4)를 MVC(Step 3) **뒤에** 두는 이유: 컨트롤러/서비스를 수동 `new` + 수동 등록하는 고통을 먼저 겪어야 IoC가 "왜" 필요한지 체감된다. 역사적 순서(서블릿 시대 → 스프링 등장)와도 일치한다.
- Step 6 → 7 순서는 필수 의존: 선언적 트랜잭션은 프록시 위에 선다.

## 4. 진행 현황

| Step | 상태 | 완료일 | tag | 문서 |
|---|---|---|---|---|
| 0 | ⬜ 대기 | - | - | [step-00-setup.md](docs/step-00-setup.md) |
| 1 | ⬜ 대기 | - | - | [step-01-http-server.md](docs/step-01-http-server.md) |
| 2 | ⬜ 대기 | - | - | [step-02-servlet-routing.md](docs/step-02-servlet-routing.md) |
| 3 | ⬜ 대기 | - | - | [step-03-annotation-mvc.md](docs/step-03-annotation-mvc.md) |
| 4 | ⬜ 대기 | - | - | [step-04-di-container.md](docs/step-04-di-container.md) |
| 5 | ⬜ 대기 | - | - | [step-05-filter-interceptor.md](docs/step-05-filter-interceptor.md) |
| 6 | ⬜ 대기 | - | - | [step-06-aop-proxy.md](docs/step-06-aop-proxy.md) |
| 7 | ⬜ 대기 | - | - | [step-07-jdbc-transaction.md](docs/step-07-jdbc-transaction.md) |
| 8 | ⬜ 대기 | - | - | [step-08-spring-porting.md](docs/step-08-spring-porting.md) |

상태: ⬜ 대기 / 🔄 진행중 / ✅ 완료

## 5. 리포지토리 구조와 규칙

```
server/
├── ROADMAP.md            # 이 문서 (허브 + 진행 현황)
├── docs/                 # 단계별 상세 문서 + 학습 노트
├── settings.gradle.kts
├── web-core/             # 직접 만드는 미니 프레임워크 (Step 1~7에서 성장)
├── app/                  # boardy 도메인 앱 (web-core 사용)
└── app-spring/           # Step 8에서 추가하는 Spring Boot 포팅본
```

규칙:
- **`web-core`에 도메인 코드 금지**: 프레임워크 모듈에 boardy 지식이 새어 들어가면 안 된다. 이 제약 자체가 Spring의 모듈 분리(spring-core / spring-web / spring-webmvc)를 체감하는 학습 장치다.
- **단계 완료 시 `git tag step-N`**: 단일 코드베이스를 계속 진화시키되 tag로 단계 스냅샷을 남긴다. `git diff step-3..step-4`가 "수동 등록 → DI 전환"의 교재가 된다.
- **DoD 충족 → 학습 노트 작성 → 태그 → 커밋** 순서를 지킨다. 학습 노트가 이 프로젝트의 실질 산출물이다.

## 6. 미니 스프링 ↔ Spring 대응표 (전체 요약)

단계별 상세 대응표는 각 step 문서 참고. 여기는 한눈표.

| 내가 만드는 것 | Spring/Tomcat의 실제 대응물 | Step |
|---|---|---|
| accept 루프 + HTTP 파서 | Tomcat Coyote (Connector, NIO Endpoint) | 1 |
| 가상 스레드 실행 모델 | `spring.threads.virtual.enabled` | 1 |
| `Handler` 인터페이스 | `jakarta.servlet.Servlet` / `HttpServlet` | 2 |
| path variable 라우터 | Tomcat `Mapper`, URL 패턴 매핑 | 2 |
| `FrontController.dispatch()` | `DispatcherServlet.doDispatch()` | 3 |
| 애노테이션 스캔 매핑 테이블 | `RequestMappingHandlerMapping` | 3 |
| `ArgumentResolver` 체인 | `HandlerMethodArgumentResolver` | 3 |
| 객체 → JSON 변환 | `HttpMessageConverter` | 3 |
| 전역 예외 처리기 | `@ControllerAdvice` / `@ExceptionHandler` | 3 |
| `BeanContainer` | `BeanFactory` / `ApplicationContext` | 4 |
| 컴포넌트 스캔 | `ClassPathBeanDefinitionScanner` | 4 |
| `Filter` + `FilterChain` | `jakarta.servlet.Filter`, `FilterChainProxy`(Security) | 5 |
| `HandlerInterceptor` | `HandlerInterceptor` + `HandlerExecutionChain` | 5 |
| `SessionStore` | `HttpSession` | 5 |
| 프록시 팩토리 + 후처리 훅 | `BeanPostProcessor`, `AnnotationAwareAspectJAutoProxyCreator` | 6 |
| `@Around` 어드바이스 | `@Aspect` / `@Around` | 6 |
| `MyJdbcTemplate` + `RowMapper` | `JdbcTemplate` + `RowMapper` | 7 |
| `ThreadLocal<Connection>` 동기화 | `TransactionSynchronizationManager` | 7 |
| `@MyTransactional` 프록시 | `TransactionInterceptor` + `@Transactional` | 7 |

## 7. 부록

- [이후 확장 주제](docs/appendix-next-topics.md) — JPA/Hibernate, WebFlux/Netty, HikariCP, Spring Security 아키텍처 등. 이 로드맵은 끝이 아니라 다음 지도의 출발점이다.
