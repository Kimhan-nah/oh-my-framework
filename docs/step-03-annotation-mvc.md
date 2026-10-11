# Step 3 — 애노테이션 기반 MVC 프레임워크

## 학습 목표

- DispatcherServlet이 사실상 **"리플렉션 기반 라우팅 + 파라미터 바인딩 + 반환값 변환"의 조합**임을 구현으로 증명한다.
- Spring MVC 요청 처리 파이프라인의 전체 그림을 그릴 수 있게 된다.

## 실무 미스터리

- `@GetMapping` 붙은 메서드는 누가, 언제, 어떻게 찾아서 호출해주는 걸까?
- `@RequestBody`로 받은 DTO는 어디서 만들어지나? `@PathVariable`은 어떻게 String에서 Long으로 변환되나?

## 구현할 것

- [ ] 애노테이션 정의: `@Controller`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PathVariable`, `@RequestParam`, `@RequestBody`
- [ ] `FrontController` (단일 진입 Handler — Step 2 라우터를 대체)
- [ ] `HandlerMapping`: 컨트롤러 클래스의 애노테이션을 스캔해 `(HTTP 메서드, 경로 패턴) → Method` 매핑 테이블 구성
- [ ] `HandlerAdapter`: 리플렉션으로 대상 메서드 invoke
- [ ] `ArgumentResolver` 체인: path variable / 쿼리 파라미터 / JSON 바디 → 메서드 파라미터 바인딩 (data class DTO)
  - JSON 직렬화/역직렬화는 **Jackson 사용** — JSON 파싱은 이 프로젝트의 학습 목표가 아니다
- [ ] 반환값 처리: 객체 → JSON 응답 (`MessageConverter` 흉내)
- [ ] 전역 예외 처리기 1개 (`@ExceptionHandler` 스타일): 잘못된 JSON → 400
- [ ] **boardy v2**: 핸들러 람다를 전부 `@Controller` 클래스로 리팩터링
  - 단, 컨트롤러 인스턴스는 아직 **수동 `new` + 수동 등록** — 이 고통이 Step 4(DI)의 동기다

## Spring 대응표

| 내가 만드는 것 | Spring의 대응물 |
|---|---|
| `FrontController.dispatch()` | `DispatcherServlet.doDispatch()` |
| 애노테이션 스캔 매핑 테이블 | `RequestMappingHandlerMapping` |
| 리플렉션 invoke | `RequestMappingHandlerAdapter` + `InvocableHandlerMethod` |
| `ArgumentResolver` 체인 | `HandlerMethodArgumentResolver` |
| 객체 → JSON | `HttpMessageConverter` (`MappingJackson2HttpMessageConverter`) |
| 전역 예외 처리기 | `@ControllerAdvice` / `@ExceptionHandler`, `HandlerExceptionResolver` |

**비교 포인트**: 내 `dispatch()` 흐름을 Spring `DispatcherServlet.doDispatch()` 소스와 나란히 놓고 단계별 대응표를 작성한다 (학습 노트에 표로 기록). Step 8 디버거 추적의 예습이 된다.

## 완료 기준 (DoD)

- [ ] `curl -X POST http://localhost:8080/posts -H 'Content-Type: application/json' -d '{"title":"t","content":"c"}'` → 201 + JSON 응답
- [ ] `@PathVariable`로 `GET /posts/1` 동작 (String → Long 변환 포함)
- [ ] 잘못된 JSON 바디 → 전역 예외 처리기가 400 반환
- [ ] **컨트롤러를 하나 추가할 때 `web-core` 코드 수정 0줄** (등록 코드는 `app`에만)

## 심화 과제 (Optional)

- Content negotiation (`Accept` 헤더에 따라 JSON / plain text 분기)
- `@RequestParam(required = false, defaultValue = ...)` 지원
- 커스텀 ArgumentResolver를 사용자가 등록할 수 있는 확장점 설계

## 권장 학습 자료 키워드

- Spring Framework 공식 문서 "Web MVC > DispatcherServlet > Processing"
- 토비의 스프링 Vol.2 — 스프링 MVC 장
- Java Reflection API (`Method.invoke`, `Parameter.getName`) — Kotlin에서 그대로 사용 가능
- kotlin-reflect (`KFunction`, `KParameter.name`) — Java 리플렉션만으로 파라미터 이름이 안 나올 때(`-java-parameters` 컴파일 옵션)와 비교해볼 것
- Jackson `jackson-module-kotlin` (data class 역직렬화)

## 학습 노트

(진행하며 기록 — doDispatch 대응표 포함)

## 회고: Spring이라면?

(완료 후 작성)
