# Spring Boot CRUD 프로젝트

Spring Boot 기반의 RESTful CRUD API 프로젝트입니다.  
**JPA**와 **MyBatis**를 동시에 사용하여 두 ORM/SQL Mapper의 차이를 비교 학습할 수 있으며, **Docker**를 통한 컨테이너 배포까지 전 과정을 다룹니다.

---

## 기술 스택

| 분류 | 기술 | 버전 |
|------|------|------|
| Language | Java | 17 |
| Framework | Spring Boot | 3.4.2 |
| ORM | Spring Data JPA (Hibernate) | Spring Boot 관리 |
| SQL Mapper | MyBatis | 3.0.4 |
| Database | MySQL | 8.0 |
| Build Tool | Gradle (Kotlin DSL) | - |
| Container | Docker / Docker Compose | - |
| Utility | Lombok | - |
| Validation | Spring Boot Starter Validation | - |

---

## 기술 선택 이유

### Spring Boot
- 자동 설정(Auto Configuration)으로 빠른 프로젝트 셋업 가능
- 내장 Tomcat 서버로 별도 WAS 설정 불필요
- Spring 생태계(JPA, MyBatis, Validation 등)와의 높은 통합성

### JPA (Spring Data JPA + Hibernate)
- 객체 중심 개발: SQL 없이 엔티티 객체만으로 DB 조작 가능
- `ddl-auto: update`로 엔티티 변경 시 스키마 자동 반영
- 복잡한 조인 없이 단순 CRUD에 최적화된 Repository 패턴 제공
- 비교 학습 목적: MyBatis와의 개발 방식 차이 체험

### MyBatis
- SQL을 직접 작성하여 쿼리 최적화 및 세밀한 제어 가능
- 복잡한 쿼리나 레거시 DB 환경에 유리
- 비교 학습 목적: JPA와 동일한 CRUD를 SQL 방식으로 구현
- `@Mapper` 애노테이션 기반의 직관적인 인터페이스 구성

### MySQL
- 국내외 가장 널리 사용되는 관계형 데이터베이스
- Spring Boot + JPA + MyBatis와의 높은 호환성
- Docker 공식 이미지를 통한 간편한 컨테이너 실행

### Gradle (Kotlin DSL)
- Groovy DSL 대비 타입 안전성과 IDE 자동완성 지원 향상
- 빠른 빌드 속도와 증분 빌드(Incremental Build) 지원

### Lombok
- `@Getter`, `@Setter`, `@RequiredArgsConstructor` 등으로 보일러플레이트 코드 제거
- 생성자 주입 방식의 의존성 주입을 간결하게 표현

### Docker / Docker Compose
- 환경 차이(로컬 vs 운영)에 관계없이 동일한 실행 환경 보장
- MySQL 컨테이너와 Spring App 컨테이너를 `depends_on`으로 의존성 관리
- 볼륨 마운트로 DB 데이터 영속성 확보

---

## 프로젝트 구조

```
Spring-CRUD-main/
├── src/
│   ├── main/
│   │   ├── java/com/example/crud/
│   │   │   ├── CrudDemoApplication.java      # 애플리케이션 진입점
│   │   │   ├── controller/
│   │   │   │   └── ItemController.java       # REST API 엔드포인트 정의
│   │   │   ├── service/
│   │   │   │   └── ItemService.java          # 비즈니스 로직 (JPA / MyBatis 분리)
│   │   │   ├── repository/
│   │   │   │   └── ItemRepository.java       # JPA Repository 인터페이스
│   │   │   ├── mapper/
│   │   │   │   └── ItemMapper.java           # MyBatis Mapper 인터페이스 (애노테이션 SQL)
│   │   │   └── entity/
│   │   │       └── Item.java                 # JPA 엔티티 / DB 테이블 매핑
│   │   └── resources/
│   │       ├── application.yml               # 공통 설정 (활성 프로파일 지정)
│   │       ├── application-local.yaml        # 로컬 실행 설정 (포트 8080)
│   │       └── application-docker.yaml       # Docker 실행 설정 (포트 8081)
│   └── test/
│       └── java/com/example/crud/
│           └── CrudDemoApplicationTests.java
├── Dockerfile                                # Spring App 컨테이너 빌드 설정
├── docker-compose.yaml                       # MySQL + Spring App 컨테이너 구성
├── build.gradle.kts                          # Gradle 빌드 스크립트 (Kotlin DSL)
└── settings.gradle.kts
```

### 레이어 구조

```
Controller  →  Service  →  Repository (JPA)
                       →  Mapper     (MyBatis)
                                ↓
                             MySQL DB
```

---

## 데이터 모델

### Item 엔티티

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | Long | PK, AUTO_INCREMENT |
| `name` | String | 아이템 이름 |
| `price` | int | 아이템 가격 |

---

## API 엔드포인트

Base URL (로컬): `http://localhost:8080`  
Base URL (Docker): `http://localhost:8081`

### JPA 기반 API

| 메서드 | 엔드포인트 | 설명 | 요청 Body | 응답 |
|--------|-----------|------|-----------|------|
| `POST` | `/items/jpa` | 아이템 생성 | `{"name": "...", "price": 0}` | 생성된 `Item` 객체 |
| `GET` | `/items/jpa` | 전체 아이템 조회 | - | `Item[]` 배열 |
| `GET` | `/items/jpa/{id}` | 특정 아이템 조회 | - | `Item` 객체 or 404 |
| `PUT` | `/items/jpa/{id}` | 아이템 수정 | `{"name": "...", "price": 0}` | 수정된 `Item` 객체 or 404 |
| `DELETE` | `/items/jpa/{id}` | 아이템 삭제 | - | 204 No Content |

### MyBatis 기반 API

| 메서드 | 엔드포인트 | 설명 | 요청 Body | 응답 |
|--------|-----------|------|-----------|------|
| `POST` | `/items/mybatis` | 아이템 생성 | `{"name": "...", "price": 0}` | `"Item saved with MyBatis"` |
| `GET` | `/items/mybatis` | 전체 아이템 조회 | - | `Item[]` 배열 |
| `GET` | `/items/mybatis/{id}` | 특정 아이템 조회 | - | `Item` 객체 or 404 |
| `PUT` | `/items/mybatis/{id}` | 아이템 수정 | `{"name": "...", "price": 0}` | `"Item updated with MyBatis"` |
| `DELETE` | `/items/mybatis/{id}` | 아이템 삭제 | - | 204 No Content |

### 요청 예시

```json
POST /items/jpa
Content-Type: application/json

{
  "name": "노트북",
  "price": 1500000
}
```

---

## 실행 방법

### 로컬 실행 (MySQL 별도 설치 필요)

1. MySQL에서 데이터베이스 생성
    ```sql
    CREATE DATABASE crud_demo;
    ```

2. `application-local.yaml`에서 DB 접속 정보 확인/수정
    ```yaml
    spring:
      datasource:
        url: jdbc:mysql://localhost:3306/crud_demo?serverTimezone=UTC&characterEncoding=UTF-8
        username: root
        password:
    ```

3. 애플리케이션 실행
    ```bash
    ./gradlew bootRun
    ```

4. API 접속: `http://localhost:8080`

---

### Docker 실행 (MySQL 포함, 권장)

1. JAR 파일 빌드
    ```bash
    ./gradlew build
    ```

2. Docker Compose로 실행
    ```bash
    docker-compose up --build
    ```

3. API 접속: `http://localhost:8081`

#### Docker 구성 요약

| 서비스 | 이미지 | 내부 포트 | 외부 포트 |
|--------|--------|-----------|-----------|
| `db` | mysql:8.0 | 3306 | 3306 |
| `app` | 빌드된 JAR | 8081 | 8081 |

---

## 환경 설정 비교

| 항목 | 로컬 (`application-local.yaml`) | Docker (`application-docker.yaml`) |
|------|--------------------------------|-------------------------------------|
| DB Host | `localhost` | `db` (컨테이너 서비스명) |
| DB Password | (없음) | `password` |
| Server Port | `8080` | `8081` |
| 활성 프로파일 | `local` | `docker` |

---

## JPA vs MyBatis 비교

| 항목 | JPA | MyBatis |
|------|-----|---------|
| SQL 작성 | 불필요 (자동 생성) | 직접 작성 |
| 학습 난이도 | 높음 (ORM 개념 필요) | 낮음 (SQL 지식으로 충분) |
| 쿼리 최적화 | 제한적 | 세밀한 제어 가능 |
| 스키마 자동화 | `ddl-auto`로 가능 | 수동 관리 |
| 적합한 상황 | 도메인 중심 개발 | 복잡한 쿼리, 레거시 DB |
