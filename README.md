[README.md](https://github.com/user-attachments/files/31767608/README.md)
# Voca Project

Spring Boot와 MySQL로 구성된 백엔드 프로젝트의 초기 골격입니다. 현재 웹 API, 입력값 검증, JPA, Lombok을 사용할 수 있는 개발 환경과 Docker Compose 기반 로컬 데이터베이스가 준비되어 있습니다.

## 현재 구현 상태

- Spring Boot 애플리케이션 진입점 구성
- Spring Web, Validation, Spring Data JPA 의존성 설정
- MySQL 8 로컬 개발 환경 구성
- JUnit 기반 애플리케이션 컨텍스트 테스트 구성

아직 도메인 모델, Repository, Service, Controller와 실제 HTTP API는 구현되어 있지 않습니다.

## 기술 스택

| 구분 | 기술 |
| --- | --- |
| 언어 | Java 17 |
| 프레임워크 | Spring Boot 3.5.6 |
| 빌드 | Gradle 8.14.3, Gradle Wrapper |
| 웹 | Spring Web |
| 데이터 | Spring Data JPA, MySQL Connector/J |
| 검증 | Jakarta Bean Validation |
| 개발 편의 | Lombok |
| 테스트 | JUnit 5, Spring Boot Test |
| 로컬 인프라 | Docker Compose, MySQL 8 |

## 프로젝트 구조

```text
.
├── build.gradle
├── settings.gradle
├── docker-compose.yml
├── gradlew
├── gradlew.bat
├── gradle/wrapper/
└── src/
    ├── main/
    │   ├── java/com/example/gongnamul_project/
    │   │   └── GongnamulProjectApplication.java
    │   └── resources/
    │       └── application.properties
    └── test/java/com/example/gongnamul_project/
        └── GongnamulProjectApplicationTests.java
```

## 시작하기

### 요구 사항

- JDK 17
- Docker 및 Docker Compose

Gradle은 Wrapper가 포함되어 있으므로 별도로 설치하지 않아도 됩니다.
프로젝트의 컴파일 대상도 Java 17로 설정되어 있으므로 Gradle 실행 시 JDK 17을 사용하는 것을 권장합니다.

### 1. 저장소 복제

```bash
git clone https://github.com/hyunI0326/voca_project.git
cd voca_project
```

### 2. MySQL 실행

```bash
docker compose up -d db
```

기본 로컬 데이터베이스 정보는 다음과 같습니다.

| 항목 | 값 |
| --- | --- |
| 호스트 | `localhost` |
| 포트 | `3307` |
| 데이터베이스 | `vocab` |
| 사용자 | `vocab` |
| 비밀번호 | `vocabpass` |
| root 비밀번호 | `pass` |

컨테이너 상태와 로그는 다음 명령으로 확인할 수 있습니다.

```bash
docker compose ps
docker compose logs -f db
```

### 3. 애플리케이션 실행

현재 `application.properties`에는 데이터베이스 접속 정보가 없으므로 환경 변수로 전달해야 합니다.

macOS/Linux:

```bash
SPRING_DATASOURCE_URL='jdbc:mysql://localhost:3307/vocab?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul' \
SPRING_DATASOURCE_USERNAME='vocab' \
SPRING_DATASOURCE_PASSWORD='vocabpass' \
./gradlew bootRun
```

Windows PowerShell:

```powershell
$env:SPRING_DATASOURCE_URL='jdbc:mysql://localhost:3307/vocab?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul'
$env:SPRING_DATASOURCE_USERNAME='vocab'
$env:SPRING_DATASOURCE_PASSWORD='vocabpass'
.\gradlew.bat bootRun
```

서버는 기본적으로 `http://localhost:8080`에서 실행됩니다. 현재 등록된 Controller가 없으므로 루트 주소에 접속하면 `404 Not Found`가 반환되는 것이 정상입니다.

### 4. 테스트 실행

컨텍스트 테스트도 데이터베이스 연결이 필요합니다. MySQL 컨테이너가 실행 중인 상태에서 다음 명령을 사용합니다.

```bash
SPRING_DATASOURCE_URL='jdbc:mysql://localhost:3307/vocab?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul' \
SPRING_DATASOURCE_USERNAME='vocab' \
SPRING_DATASOURCE_PASSWORD='vocabpass' \
./gradlew test
```

테스트 결과는 `build/reports/tests/test/index.html`에서 확인할 수 있습니다.

### 5. 개발 환경 종료

애플리케이션을 종료한 뒤 MySQL 컨테이너를 중지합니다.

```bash
docker compose down
```

`db_data` 볼륨은 유지되므로 다음 실행에서도 데이터가 보존됩니다. 데이터까지 초기화하려면 `docker compose down -v`를 사용하세요.

## 설정 참고

Spring Boot의 환경 변수는 다음 설정 항목에 대응합니다.

| 환경 변수 | Spring 설정 |
| --- | --- |
| `SPRING_DATASOURCE_URL` | `spring.datasource.url` |
| `SPRING_DATASOURCE_USERNAME` | `spring.datasource.username` |
| `SPRING_DATASOURCE_PASSWORD` | `spring.datasource.password` |

엔티티를 추가한 뒤 개발 환경에서 Hibernate가 스키마를 갱신하게 하려면 실행 시 아래 환경 변수를 추가할 수 있습니다.

```bash
SPRING_JPA_HIBERNATE_DDL_AUTO=update
```

운영 환경에서는 `ddl-auto=update` 대신 Flyway 또는 Liquibase 같은 마이그레이션 도구를 사용하는 것을 권장합니다.

## 개발 시 확인할 점

- `docker-compose.yml`의 비밀번호는 로컬 개발용입니다. 운영 환경에서는 저장소에 비밀번호를 커밋하지 말고 Secret 또는 환경 변수로 관리하세요.
- 호스트에서는 MySQL에 `3307` 포트로 연결하지만, Docker 네트워크 내부의 다른 서비스에서는 `db:3306`을 사용합니다.
- API를 추가하면 Controller 테스트와 Service 단위 테스트도 함께 작성하는 것이 좋습니다.
- 패키지명과 애플리케이션명은 현재 `gongnamul_project`입니다. 저장소 이름인 `voca_project`와 통일하려면 리팩터링이 필요합니다.

## 라이선스

현재 저장소에는 별도의 라이선스가 명시되어 있지 않습니다.
