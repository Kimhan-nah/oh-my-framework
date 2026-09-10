# Step 7 — JDBC 추상화 + 선언적 트랜잭션

## 학습 목표

- JdbcTemplate이 없애주는 보일러플레이트의 정체(자원 해제, 예외 변환)를 고통으로 체험한 뒤 직접 추출한다.
- **트랜잭션 동기화** — 서비스 계층 전체가 같은 커넥션을 공유하는 메커니즘(= ThreadLocal)을 구현한다.
- `@Transactional`이 **AOP 프록시 + 커넥션 동기화의 결합**임을 구현으로 증명한다.

## 실무 미스터리

- **서비스에서 리포지토리 두 개를 호출해도 어떻게 같은 커넥션(같은 트랜잭션)을 쓸까?** — 실무자 대부분이 설명 못 하는 지점.
- 왜 런타임 예외만 롤백이 기본일까? checked 예외는 왜 커밋될까?

## 구현할 것

- [ ] H2(in-memory) + **순수 JDBC로 boardy 리포지토리 1개를 먼저 작성** (Connection/PreparedStatement/ResultSet 수동 관리 — 고통 체험이 목적)
- [ ] `MyJdbcTemplate` 추출:
  - [ ] `query(sql, rowMapper, args...)` / `queryForObject(...)` / `update(sql, args...)`
  - [ ] `RowMapper<T>` 함수형 인터페이스
  - [ ] checked `SQLException` → unchecked 예외 변환
- [ ] `TransactionManager`:
  - [ ] `ThreadLocal<Connection>` 기반 커넥션 바인딩 (트랜잭션 중이면 같은 커넥션 반환)
  - [ ] `MyJdbcTemplate`이 커넥션을 직접 열지 않고 매니저에게 받도록 연결
- [ ] `@MyTransactional` + Step 6 프록시로 선언적 트랜잭션: begin(auto-commit off) → 성공 시 commit / 예외 시 rollback → 커넥션 정리
- [ ] **핵심 시나리오**: 글 작성(post insert) + 포인트 적립(point update)을 한 트랜잭션으로 묶고, 포인트 로직에서 강제 예외 시 글도 롤백되는지 검증
- [ ] boardy 저장소 전체를 in-memory Map → H2로 교체 (**컨트롤러/서비스 코드 무변경** = 계층 분리 검증)
- [ ] (학습 노트 주제) 가상 스레드 + ThreadLocal 커넥션의 궁합 문제, `ScopedValue`(JEP 446) — Step 1에서 선택한 Java 21의 회수 지점

## Spring 대응표

| 내가 만드는 것 | Spring의 대응물 |
|---|---|
| `MyJdbcTemplate` | `JdbcTemplate` |
| `RowMapper` | `RowMapper` |
| SQLException 변환 | `DataAccessException` 계층 (`SQLExceptionTranslator`) |
| `TransactionManager` | `DataSourceTransactionManager` |
| `ThreadLocal<Connection>` 바인딩 | `TransactionSynchronizationManager` + `DataSourceUtils.getConnection()` |
| `@MyTransactional` 프록시 | `TransactionInterceptor` + `@Transactional` |

**비교 포인트**:
- 내 `TransactionManager`의 ThreadLocal ↔ Spring의 `TransactionSynchronizationManager` — "선언적 트랜잭션의 마법"의 실체가 ThreadLocal 하나임을 확인.
- 롤백 규칙(런타임 예외만 롤백)을 내가 직접 결정해보고 Spring의 기본값과 비교 — 그 선택의 트레이드오프를 설명할 수 있게 된다.

## 완료 기준 (DoD)

- [ ] in-memory Map 리포지토리 전부 H2로 교체 완료, 컨트롤러/서비스 코드 변경 0줄
- [ ] **포인트 적립 실패 시 글 작성도 롤백되는 통합 테스트 통과**
- [ ] `@MyTransactional` 없는 메서드는 auto-commit으로 동작함을 테스트로 확인
- [ ] 같은 트랜잭션 안에서 두 리포지토리가 동일 커넥션을 쓰는지 확인하는 테스트 (커넥션 identity 비교)

## 심화 과제 (Optional)

- 전파(propagation) 흉내: `REQUIRED`(기본) / `REQUIRES_NEW` 두 가지만 구현해보기
- 읽기 전용 트랜잭션 (`readOnly=true` → `connection.setReadOnly`)
- 간단한 커넥션 풀 구현 후 HikariCP와 개념 비교
- `ScopedValue`로 ThreadLocal 대체 실험 (가상 스레드 친화)

## 권장 학습 자료 키워드

- 토비의 스프링 Vol.1 — 3장 (템플릿), 5장 (서비스 추상화)
- Spring 공식 문서 "Transaction Management"
- JDBC Connection / auto-commit
- JEP 446 Scoped Values

## 학습 노트

(진행하며 기록 — 가상 스레드 × ThreadLocal 정리 포함)

## 회고: Spring이라면?

(완료 후 작성)
