# Step 8 — Spring Boot 포팅 & 최종 비교

## 학습 목표

- 지금까지 만든 모든 것의 "진짜 버전"을 소스/디버거 레벨에서 추적한다.
- Spring Boot 자동 설정이 감추고 있는 것들을 목록화한다.
- **행위 동등성**: 내 미니 스프링과 실제 Spring Boot가 같은 테스트를 통과하는 것이 이 프로젝트의 최종 결과물.

## 실무 미스터리

- `@SpringBootApplication` 한 줄로 서버가 뜨는 동안 실제로 무슨 일이 일어나나?
- 의존성만 추가하면 설정 없이 동작하는 자동 설정(auto-configuration)은 어떤 원리인가?

## 구현할 것

- [ ] 새 모듈 `app-spring/`: 동일한 boardy API를 Spring Boot 3.x (Kotlin)로 재작성
  - `kotlin("plugin.spring")`(all-open)을 붙이는 순간, Step 6에서 겪은 `final` 클래스 문제가 어떻게 사라지는지 확인
  - Web MVC + Spring `JdbcTemplate` + `@Transactional` + `HandlerInterceptor` 인증
  - 도메인 코드는 최대한 복사하고 **프레임워크 접점만 교체** (diff가 곧 비교 자료)
  - JPA는 의도적으로 제외 — Spring `JdbcTemplate`을 써야 Step 7과 1:1 비교가 된다 (JPA는 [부록](appendix-next-topics.md) 주제)
- [ ] **디버거 추적 세션 3회** (각각 학습 노트로 기록):
  - [ ] ① `DispatcherServlet.doDispatch()`에 브레이크포인트 → 요청 1건 완주 (Step 3과 비교)
  - [ ] ② `AbstractApplicationContext.refresh()` → 부팅 과정 추적 (Step 4와 비교)
  - [ ] ③ `TransactionInterceptor.invoke()` → 트랜잭션 시작/커밋 추적 (Step 6~7과 비교)
- [ ] 비교 문서 작성 (이 파일의 학습 노트 또는 별도 섹션):
  - [ ] "내 클래스 ↔ Spring 클래스" 최종 대응표 완성 (ROADMAP 6절 갱신)
  - [ ] "Spring이 추가로 하는 일" 목록화 — 예: 매핑 캐싱, 이벤트 발행, `Environment` 추상화, 자동 설정 조건 평가(`@ConditionalOn...`)
- [ ] (선택 확장) `spring.threads.virtual.enabled=true`로 Step 1 벤치마크 재현 비교

## Spring 대응표

전 단계의 대응표 전체 + 이 단계에서 새로 만나는 것:

| 개념 | Spring Boot의 대응물 |
|---|---|
| 내 main + 컨테이너 부팅 | `SpringApplication.run()` + `@SpringBootApplication` |
| (없음 — 수동 설정) | `@EnableAutoConfiguration`, `AutoConfiguration.imports` |
| Step 1 소켓 서버 | 내장 Tomcat 부트스트랩 (`TomcatServletWebServerFactory`) |

## 완료 기준 (DoD)

- [ ] **동일한 HTTP 통합 테스트 스위트(또는 `test.http`/curl 스크립트)가 `app`(미니 스프링)과 `app-spring` 양쪽에서 통과**
- [ ] 디버거 추적 노트 3건 작성 완료
- [ ] 최종 대응표 + "Spring이 추가로 하는 일" 목록 작성 완료
- [ ] ROADMAP.md 진행 현황표 전체 ✅, `git tag step-8`

## 심화 과제 (Optional)

- 자동 설정 직접 작성: `web-core`를 Spring Boot starter처럼 포장해보기 (`AutoConfiguration.imports`)
- Actuator를 붙여 내 미니 스프링에는 없는 운영 기능 목록 확인
- 두 앱의 부팅 시간/메모리 사용량 비교

## 권장 학습 자료 키워드

- Spring Boot Reference — "Developing with Spring Boot", "Creating Your Own Auto-configuration"
- spring-framework GitHub 소스 (`DispatcherServlet`, `AbstractApplicationContext`, `TransactionInterceptor`)
- How Spring Boot auto-configuration works

## 학습 노트

(디버거 추적 노트 3건 + 최종 비교표)

## 회고: Spring이라면?

이 단계 자체가 회고다 — 프로젝트 전체를 마친 뒤, "처음에 몰랐지만 지금은 설명할 수 있는 것" 목록을 여기에 남긴다.
