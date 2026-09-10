# Step 2 — 서블릿 스타일 추상화 + 라우팅

## 학습 목표

- "왜 서블릿이라는 추상화가 필요한가"를 체득한다 — 소켓/파싱 세부를 감추고 요청 처리 계약(interface)만 남기는 과정.
- 서블릿 컨테이너가 하는 일의 최소 집합을 직접 정의해본다.

## 실무 미스터리

- 서블릿 URL 패턴(`/posts/*`)으로는 왜 `/posts/{id}/comments` 같은 매핑이 어색해질까? (→ 프론트 컨트롤러가 필요한 이유, Step 3의 동기)

## 구현할 것

- [ ] `Handler` 인터페이스 (≒ Servlet): `HttpResponse handle(HttpRequest request)`
- [ ] 경로 + 메서드 기반 라우터: `router.get("/posts/{id}", handler)`
- [ ] **path variable 매칭 직접 구현** (`/posts/{id}` ↔ `/posts/42` → `id=42` 추출)
- [ ] 404(경로 없음) / 405(메서드 미지원) / 500(핸들러 예외) 기본 처리
- [ ] 쿼리스트링 파싱, 폼(form-urlencoded) 바디 파싱
- [ ] **boardy v1**: in-memory `Map` 저장소로 users/posts CRUD를 핸들러 람다로 등록

## Spring 대응표

| 내가 만드는 것 | Spring/Tomcat의 대응물 |
|---|---|
| `Handler` 인터페이스 | `jakarta.servlet.Servlet` / `HttpServlet.service()` |
| `HttpRequest` / `HttpResponse` | `HttpServletRequest` / `HttpServletResponse` |
| 라우터 (경로→핸들러 테이블) | Tomcat `Mapper` (URL → 서블릿 매핑) |
| `router.get(...)` 등록 | `web.xml` / `ServletRegistration` |

**비교 포인트**: 내 라우터의 path variable 매칭 ↔ 서블릿 URL 패턴(exact / prefix / extension)의 한계. 이 한계가 "서블릿 하나가 모든 요청을 받고(front controller) 그 안에서 정교하게 라우팅"하는 Spring MVC 구조의 탄생 배경이다.

## 완료 기준 (DoD)

- [ ] `curl -X POST http://localhost:8080/users -d 'name=hannie'` → 201, `curl http://localhost:8080/users/1` → JSON 문자열 반환
- [ ] 존재하지 않는 경로 → 404, 등록 안 된 메서드 → 405
- [ ] 핸들러에서 예외 발생 시 → 500 (서버는 죽지 않음)
- [ ] 라우터 매칭 단위 테스트 통과 (정확 매칭 / path variable / 매칭 실패)

## 심화 과제 (Optional)

- 라우팅 우선순위: `/posts/new`(정확)와 `/posts/{id}`(변수)가 둘 다 등록됐을 때 어느 쪽이 이겨야 하나 — 규칙을 정하고 테스트로 고정
- trie 기반 라우터로 개선해보기 (선형 탐색 vs trie 성능 비교)

## 권장 학습 자료 키워드

- Jakarta Servlet Specification 6.0 (1~2장)
- Front Controller pattern (마틴 파울러, PoEAA)
- Tomcat request processing flow

## 학습 노트

(진행하며 기록)

## 회고: Spring이라면?

(완료 후 작성)
