# Step 5 — Filter / Interceptor 체인 + 인증

## 학습 목표

- 서블릿 필터(프레임워크 **밖**, 요청/응답 레벨)와 인터셉터(프레임워크 **안**, 핸들러 레벨)의 구조적 차이를 이해한다.
- 체인 패턴(chain-of-responsibility)과 `chain.doFilter()`의 재귀 구조를 직접 구현한다.
- 인증이 왜 필터 계층에 놓이는지(Spring Security의 위치) 이해한다.

## 실무 미스터리

- **왜 필터에서 던진 예외는 `@ControllerAdvice`가 못 잡을까?** → 내 구현에서 직접 재현한다.
- 인터셉터의 `postHandle`은 왜 예외가 나면 실행이 스킵되나? (`afterCompletion`과의 차이)

## 구현할 것

- [ ] `Filter` 인터페이스 + `FilterChain` — `chain.next(request)` 호출 구조(chain-of-responsibility) 직접 구현
- [ ] `HandlerInterceptor`: `preHandle` / `postHandle` / `afterCompletion` — `FrontController` 파이프라인 안에 삽입
- [ ] 로깅 필터 (요청/응답/소요시간)
- [ ] CORS 필터 (간단 버전)
- [ ] **인증 구현**:
  - [ ] `POST /login` → 세션 ID 발급, in-memory `SessionStore`, `Set-Cookie`
  - [ ] 인증 필터: 쓰기 API에서 세션 쿠키 검증, 미인증 시 401
  - [ ] `@LoginRequired` 애노테이션 + 인터셉터로 선언적 인증 (핸들러 메서드의 애노테이션 읽기)
  - [ ] `@AuthUser` ArgumentResolver: 로그인 사용자를 컨트롤러 파라미터로 주입
- [ ] 필터/인터셉터를 빈으로 등록 (Step 4 컨테이너 활용)

## Spring 대응표

| 내가 만드는 것 | Spring의 대응물 |
|---|---|
| `Filter` + `FilterChain` | `jakarta.servlet.Filter` / `FilterChain` |
| 인증 필터 | Spring Security `FilterChainProxy` / `SecurityFilterChain` (구조 개념 비교) |
| `HandlerInterceptor` | `HandlerInterceptor` + `HandlerExecutionChain` |
| `SessionStore` + 쿠키 | `HttpSession` + `JSESSIONID` |
| `@AuthUser` resolver | `@AuthenticationPrincipal` 류의 커스텀 ArgumentResolver |

**비교 포인트**: 실행 위치가 곧 능력의 차이다.

```
[클라이언트]
   ↓
Filter 체인          ← 프레임워크(FrontController) 밖. 예외 나면 전역 처리기 도달 불가
   ↓
FrontController
   ├─ Interceptor.preHandle
   ├─ 핸들러(컨트롤러) 실행   ← 예외는 여기서 전역 처리기가 잡음
   ├─ Interceptor.postHandle  ← 예외 시 스킵
   └─ Interceptor.afterCompletion ← 예외여도 실행
   ↓
Filter 체인 (응답 역순)
```

## 완료 기준 (DoD)

- [ ] 미로그인 `POST /posts` → 401
- [ ] `POST /login` → 쿠키 발급 → 쿠키 포함 `POST /posts` → 201
- [ ] 필터/인터셉터 실행 순서가 로그로 확인 가능 (요청: F1→F2→pre→핸들러, 응답: post→after→F2→F1)
- [ ] **필터에서 예외 발생 시 전역 예외 처리기가 적용되지 않음을 테스트로 증명**
- [ ] 핸들러 예외 시 `postHandle` 스킵 + `afterCompletion` 실행을 테스트로 증명

## 심화 과제 (Optional)

- 인터셉터 경로 패턴 등록 (`addPathPatterns` / `excludePathPatterns` 흉내)
- 세션 만료(TTL) 구현
- 토큰(간단 서명) 방식으로 인증 교체해보고 세션 방식과 비교

## 권장 학습 자료 키워드

- Servlet Filter (chain-of-responsibility)
- Spring 공식 문서 "HandlerInterceptor"
- Spring Security Architecture (공식 topical guide — FilterChainProxy 그림)
- HTTP cookies & session management

## 학습 노트

(진행하며 기록)

## 회고: Spring이라면?

(완료 후 작성)
