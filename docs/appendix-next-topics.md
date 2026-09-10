# 부록 — 이후 확장 주제

이 로드맵(Step 0~8)은 끝이 아니라 다음 지도의 출발점이다. Step 8까지 완료한 뒤 흥미에 따라 골라 진행한다.

## 후보 주제

### JPA / Hibernate 내부 원리
- 이 로드맵에서 JPA를 의도적으로 뺀 이유: Spring `JdbcTemplate`과의 1:1 비교(Step 7↔8)를 유지하기 위해서. ORM은 별도의 큰 산이다.
- 미니 구현 아이디어: 영속성 컨텍스트(1차 캐시), 더티 체킹, 지연 로딩 프록시를 직접 만들어보기.
- 키워드: EntityManager, 영속성 컨텍스트, flush 시점, JPA N+1.

### 리액티브 / Netty (WebFlux)
- Step 1의 "커넥션당 스레드" 모델과 정반대인 이벤트 루프 모델. 가상 스레드(Step 1)와의 비교가 특히 좋은 논점.
- 미니 구현 아이디어: `java.nio.Selector`로 싱글 스레드 이벤트 루프 에코 서버 만들기.
- 키워드: NIO Selector, 이벤트 루프, Reactor(Mono/Flux), 백프레셔.

### 커넥션 풀 (HikariCP)
- Step 7 심화 과제의 연장. 왜 HikariCP가 빠른지(ConcurrentBag, 바이트코드 수준 최적화) 분석.
- 키워드: HikariCP internals, pool sizing 공식, connection leak detection.

### Spring Security 전체 아키텍처
- Step 5에서 개념만 본 `FilterChainProxy`를 실제로 추적. 인증(Authentication)과 인가(Authorization)의 객체 모델.
- 키워드: SecurityContextHolder, AuthenticationManager, 필터 순서.

### HTTP/2, TLS
- Step 1의 HTTP/1.1 파서가 왜 HTTP/2에서 완전히 달라지는지(바이너리 프레이밍, 멀티플렉싱).
- 키워드: RFC 9113, ALPN, HPACK.

### 그 외
- 내장 서버 만들기 심화: graceful shutdown, 헬스체크
- 메트릭/관측성: Micrometer 스타일 추상화 직접 만들기
- 애노테이션 프로세싱(APT): 리플렉션 대신 컴파일 타임 코드 생성 (Lombok/QueryDSL Q타입의 원리)
