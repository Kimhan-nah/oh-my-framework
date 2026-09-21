# Gradle
### Gradle 이란?
- JVM 기반 프로젝트를 위한 빌드 자동화 도구
- 소스 코드를 컴파일, 테스트 실행, 의존성(라이브러리) 관리, 최종적으로 실행 가능한 파일(jar, war 등)로 패키징하는 전 과정을 자동화
- 하는 일
  - 필요한 외부 라이브러리 다운로드 및 버전 관리
  - `.java` 파일들 컴파일
  - 테스트 코드 실행
  - 컴파일된 결과물을 `.jar`, `.war'로 패키징
  - 이 모든 과정을 정해진 순서(task)대로 실행

### gradle 구조
```
gradle/wrapper/
├── gradle-wrapper.jar          // Wrapper 실행용 바이너리
└── gradle-wrapper.properties   // 사용할 Gradle 버전 명시
gradlew                          // Mac/Linux용 실행 스크립트
gradlew.bat                      // Windows용 실행 스크립트
```

### Gradle 필수 파일
- `settings.gradle.kts` (루트, 1개)
  - 어떤 프로젝트/모듈들이 존재하는지 선언
  - gradle이 가장 먼저 읽는 파일 (초기화 단계)
  - 주 내용 : 프로젝트 이름, 플러그인 관리 저장소
  - 예시
    ```
    rootProject.name = "my-project"   // 전체 프로젝트 이름
    include("api")                     // 서브모듈 등록
    include("domain")
    include("infra")
    include("common")
    ```
- `build.gradle.kts` (루트, 1개 / 모듈별, 1개)
  - 각 프로젝트를 어떻게 빌드할지 선언
  - settings 다음, 각 프로젝트 설정 단계
  - 주 내용 : 의존성, 빌드 스크립트, 플러그인 적용, 빌드 옵션
  - 예시
    ```
    plugins {
        id("org.springframework.boot") version "3.1.0"
        id("io.spring.dependency-management") version "1.1.0"
        kotlin("jvm") version "1.8.10"
        kotlin("plugin.spring") version "1.8.10"
    }

    dependencies {
        implementation("org.springframework.boot:spring-boot-starter-web")
        implementation("org.jetbrains.kotlin:kotlin-reflect")
        testImplementation("org.springframework.boot:spring-boot-starter-test")
    }
    ```

### Gradle 선택 파일
- `gradle.properties` : JVM 옵션, 프로젝트 전역 프로퍼티 설정
- `gradlew`, `gradlew.bat` : Gradle Wrapper 실행 스크립트
- `gradle/wrapper/gradle-wrapper.properties` : wrapper가 사용할 Gradle 버전 지정
- `gradle/wrapper/gradle-wrapper.jar` : wrapper 실행용 JAR 바이너리
- 

### Gradle Wrapper (`gradlew`)
프로젝트에 필요한 특정 버전의 gradle을 자동으로 다운로드하고 실행해주는 스크립트
프로젝트에서 사용할 gradle 버전


### 사용 방법
```
./gradlew build
./gradlew bootRun
./gradlew test

# 최초 생성 (로컬에 Gradle이 설치되어 있어야 함)
gradle wrapper

# 특정 버전으로 생성/업그레이드
gradle wrapper --gradle-version 8.8
```

