# Step 4 — DI 컨테이너 / IoC

## 학습 목표

- 빈 생명주기(정의 스캔 → 인스턴스화 → 의존성 주입 → 초기화)를 구현자 관점에서 이해한다.
- 왜 생성자 주입이 권장되는지, 순환 참조가 왜 문제인지 "만드는 입장"에서 답할 수 있게 된다.

## 실무 미스터리

- `@Autowired`는 어떻게 동작하나? 스프링은 내 클래스들을 어떻게 "찾아서" 만들어주나?
- 순환 참조 에러(`BeanCurrentlyInCreationException`)는 왜 나고, 필드 주입으로 바꾸면 왜 돌아가나?

## 구현할 것

- [ ] 애노테이션 정의: `@Component`, `@Service`, `@Repository` (+ 기존 `@Controller`를 컴포넌트로 인정)
- [ ] `BeanContainer`:
  - [ ] 패키지 기반 컴포넌트 스캔 (클래스패스 탐색)
  - [ ] 생성자 파라미터 타입 기반 **재귀 인스턴스화** (생성자 주입만 지원)
  - [ ] 싱글턴 캐시 (같은 타입 요청 시 동일 인스턴스)
- [ ] **순환 참조 감지** → 생성 경로를 담은 명확한 에러 메시지로 실패
  - Spring의 3-level cache(조기 노출)는 구현하지 않고, 왜 존재하는지 학습 노트로 정리
- [ ] 인터페이스 타입 주입 (구현체 1개 매칭, 복수면 에러)
- [ ] **boardy v3**: 컨트롤러/서비스/리포지토리를 전부 빈으로 전환. `FrontController`가 컨테이너에서 `@Controller` 빈을 꺼내 `HandlerMapping`을 구성 — **MVC와 DI의 통합 지점**

## Spring 대응표

| 내가 만드는 것 | Spring의 대응물 |
|---|---|
| `BeanContainer` | `BeanFactory` / `ApplicationContext` |
| 컴포넌트 스캔 | `ClassPathBeanDefinitionScanner`, `@ComponentScan` |
| 빈 메타정보 (구현 시 암묵적) | `BeanDefinition` (Spring은 "정의"와 "인스턴스"를 분리한다) |
| 생성자 재귀 인스턴스화 | 생성자 주입 resolution (`AutowiredAnnotationBeanPostProcessor`) |
| 순환 참조 감지 실패 | `BeanCurrentlyInCreationException`, 3-level cache |
| 컨테이너 부팅 | `AbstractApplicationContext.refresh()` |

**비교 포인트**:
- 내 "스캔 → 생성 → 주입" ↔ Spring `refresh()`의 단계(`invokeBeanFactoryPostProcessors` → `finishBeanFactoryInitialization`).
- 필드 주입이 내 컨테이너에서 구현하기 싫은 형태(리플렉션으로 private 필드 강제 접근, 불완전한 객체 노출)임을 체감 → "생성자 주입 권장"이라는 실무 규칙의 재해석.

## 완료 기준 (DoD)

- [ ] `new BeanContainer("app.boardy")` 부팅만으로 전체 앱 기동 (main 함수에서 수동 조립 코드 제거)
- [ ] 컨테이너 단위 테스트 통과:
  - [ ] 싱글턴 보장 (두 번 꺼내도 같은 인스턴스)
  - [ ] 인터페이스 타입으로 구현체 주입
  - [ ] 순환 참조 시 경로가 담긴 예외 메시지
- [ ] `app` 모듈에 수동 `new` 코드 0줄 (테스트 제외)

## 심화 과제 (Optional)

- `@Qualifier` 흉내 (동일 인터페이스 구현체 복수일 때 이름으로 선택)
- 빈 스코프: 싱글턴 외 프로토타입 스코프 추가
- `@PostConstruct` 스타일 초기화 콜백

## 권장 학습 자료 키워드

- 토비의 스프링 Vol.1 — 1~2장 (IoC, 오브젝트와 의존관계)
- Spring 공식 문서 "The IoC Container"
- Martin Fowler, "Inversion of Control Containers and the Dependency Injection pattern"

## 학습 노트

(진행하며 기록 — 3-level cache 정리 포함)

## 회고: Spring이라면?

(완료 후 작성)
