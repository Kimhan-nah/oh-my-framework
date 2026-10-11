# Step 0 — 프로젝트 셋업 & 베이스라인

## 학습 목표

- Gradle 멀티모듈 구조를 이해하고 "프레임워크 모듈 vs 앱 모듈" 분리 감각을 잡는다.
- Kotlin JVM toolchain(21) 설정과 JUnit 5 테스트 하네스를 준비한다.

## 실무 미스터리

- 왜 Spring은 spring-core / spring-web / spring-webmvc 처럼 모듈이 잘게 나뉘어 있을까? 하나로 합치면 안 되나?

## 구현할 것

- [ ] Gradle 멀티모듈 골격: `settings.gradle.kts` + `web-core/` + `app/`
- [ ] Kotlin 2.x + JVM 21 toolchain 설정 (`kotlin("jvm")` 플러그인 + `kotlin { jvmToolchain(21) }`)
- [ ] JUnit 5 의존성 + 빈 테스트 1개로 테스트 하네스 검증
- [ ] `.gitignore` (Gradle/IDE 산출물)
- [ ] ROADMAP.md / docs 골격 커밋

## Spring 대응표

| 내가 만드는 것 | Spring의 대응물 |
|---|---|
| `web-core` / `app` 모듈 분리 | spring-core / spring-web / spring-webmvc 모듈 구조 |

> 이 단계에서 [spring-framework GitHub](https://github.com/spring-projects/spring-framework)의 모듈 목록을 한 번 훑어두면 이후 단계에서 "내가 지금 어느 모듈을 만들고 있는지" 감이 잡힌다.

## 완료 기준 (DoD)

- [ ] `./gradlew build` 성공 (빈 테스트 1개 포함)
- [ ] `git tag step-0` 생성

## 심화 과제 (Optional)

- Gradle version catalog(`libs.versions.toml`)로 의존성 버전 중앙 관리
- `web-core`가 외부 라이브러리에 의존하지 않도록 의존성 제약 걸어보기 (`api` vs `implementation`)

## 권장 학습 자료 키워드

- Gradle multi-project build
- Kotlin Gradle plugin / jvmToolchain
- JUnit 5 User Guide

## 학습 노트

(진행하며 기록)

## 회고: Spring이라면?

(완료 후 작성)
