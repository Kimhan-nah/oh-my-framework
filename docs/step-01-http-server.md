# Step 1 — 소켓 기반 HTTP/1.1 서버

## 학습 목표

- TCP 소켓 위에서 HTTP가 "그냥 텍스트 프로토콜"임을 체득한다.
- 스레드 모델(스레드-퍼-커넥션 → 스레드풀 → 가상 스레드)별 처리량 차이를 직접 측정한다.
- Keep-Alive와 Content-Length의 관계를 이해한다.

## 실무 미스터리

- "톰캣 스레드 200개"의 정체는 무엇인가? 그 200개는 언제 만들어지고 무엇을 하고 있나?
- 가상 스레드를 켜면(`spring.threads.virtual.enabled=true`) 뭐가 달라지나?

## 구현할 것

- [ ] `ServerSocket` accept 루프
- [ ] HTTP 요청 파서: 요청 라인 / 헤더 / 바디 → record `HttpRequest`
- [ ] 응답 직렬화: status line + headers + body → record `HttpResponse`
- [ ] 실행 모델 3종 스위칭 가능하게:
  - [ ] single-thread
  - [ ] `ExecutorService` 고정 스레드풀
  - [ ] `Executors.newVirtualThreadPerTaskExecutor()` (가상 스레드)
- [ ] Keep-Alive 지원 (`Connection: keep-alive`, Content-Length 기반 바디 경계)
- [ ] 잘못된 요청에 400 응답
- [ ] 간단한 정적 파일 서빙 (`/static`)

## Spring 대응표

| 내가 만드는 것 | Spring/Tomcat의 대응물 |
|---|---|
| accept 루프 | Tomcat `Connector` / Coyote의 Acceptor |
| HTTP 파서 | Coyote `Http11Processor` |
| 스레드풀 실행 모델 | Tomcat Worker 스레드풀 (`maxThreads=200`) |
| 가상 스레드 실행 모델 | Spring Boot 3.2+ `spring.threads.virtual.enabled` |

**비교 포인트**: 내 accept 루프 ↔ Tomcat의 Acceptor / Poller / Worker 3단 구조를 그림으로 나란히 비교해 학습 노트에 남긴다.

## 완료 기준 (DoD)

- [ ] `curl -v http://localhost:8080/health` → `200 OK` + 바디 응답
- [ ] `curl -v http://localhost:8080/health http://localhost:8080/health` 로 동일 커넥션 재사용(keep-alive) 확인 (`* Re-using existing connection` 로그)
- [ ] 간단 부하 테스트(`wrk` 또는 `ab`)로 고정 스레드풀 vs 가상 스레드 처리량 수치를 학습 노트에 기록
- [ ] HTTP 파서 단위 테스트 통과 (정상 요청 / 바디 있는 POST / 잘못된 요청 라인)

## 심화 과제 (Optional)

- Chunked transfer encoding 파싱
- 파서를 블로킹 I/O 그대로 두고, 커넥션당 블로킹이 가상 스레드에서 왜 문제가 안 되는지 스레드 덤프로 확인
- `SO_REUSEADDR`, backlog 등 소켓 옵션의 의미 조사

## 권장 학습 자료 키워드

- RFC 9112 (HTTP/1.1 Message Syntax and Routing)
- JEP 444 Virtual Threads
- Tomcat Architecture (Coyote / Catalina, Acceptor-Poller-Worker)
- 자바 네트워크 프로그래밍 (소켓 기초)

## 학습 노트

(진행하며 기록 — 벤치마크 수치 포함)

## 회고: Spring이라면?

(완료 후 작성 — Tomcat이 같은 문제를 어떻게 푸는지 요약)
