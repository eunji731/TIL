# Spring Boot + Supabase + IntelliJ 연동 정리

## 목차
1. [문서 목적](#문서-목적)
2. [Supabase란](#supabase란)
3. [왜 Pooler 연결을 사용하는가](#왜-pooler-연결을-사용하는가)
4. [전체 연동 구조](#전체-연동-구조)
5. [Spring 설정 파일 역할 정리](#spring-설정-파일-역할-정리)
6. [application.yaml 작성 방법](#applicationyaml-작성-방법)
7. [application-dev.yml 작성 방법](#application-devyml-작성-방법)
8. [.env 작성 방법](#env-작성-방법)
9. [.gitignore 작성 방법](#gitignore-작성-방법)
10. [IntelliJ에서 확인할 부분](#intellij에서-확인할-부분)
11. [정상 연결 확인 방법](#정상-연결-확인-방법)
12. [정리](#정리)

---

## 문서 목적
이 문서는 Spring Boot 프로젝트에서 Supabase PostgreSQL을 연동할 때 필요한 기본 개념과 설정 방법을 정리한 문서이다. 특히 아래 내용을 중심으로 정리한다.
- Supabase가 무엇인지
- Spring Boot와 Supabase를 어떻게 연결하는지
- IntelliJ에서 어떤 부분을 확인해야 하는지
- `application.yaml`, `application-dev.yml`, `.env`를 어떤 역할로 나눠서 작성하는지

---

## Supabase란
Supabase는 PostgreSQL 기반의 백엔드 서비스이다. 주요 특징은 다음과 같다.
- PostgreSQL 데이터베이스 제공
- 인증(Auth), 스토리지(Storage), API 기능 제공
- 클라우드 기반으로 빠르게 DB 환경 구성 가능
- PostgreSQL을 직접 사용하는 방식과 유사하게 JDBC 연결 가능

즉, Spring Boot 입장에서는 Supabase를 **외부 PostgreSQL DB 서버**처럼 연결해서 사용할 수 있다.

---

## 왜 Pooler 연결을 사용하는가
Supabase는 DB 연결 방식으로 direct connection과 pooler connection을 제공한다.

### 1. Direct connection
직접 DB에 붙는 방식이다. 환경에 따라 IPv6 기반 제약이 있을 수 있다.

### 2. Pooler connection
애플리케이션에서 보다 안정적으로 연결할 수 있도록 중간 pooler를 통해 붙는 방식이다. 이번 연동에서는 아래 pooler 호스트를 사용했다.
`aws-1-ap-northeast-2.pooler.supabase.com`

일반적으로 아래 포트를 사용한다.
- **5432**: session mode
- **6543**: transaction mode
Spring Boot 같은 일반 백엔드 애플리케이션에서는 보통 5432를 먼저 시도하는 편이 이해하기 쉽다.

---

## 전체 연동 구조

연동 구조는 아래처럼 이해하면 된다.
1. Spring Boot 애플리케이션 실행
2. `application.yaml`에서 dev 프로파일 활성화
3. `application-dev.yml`에서 datasource 설정 로드
4. `.env` 파일에서 실제 DB 접속 정보 읽기
5. Supabase PostgreSQL(pooler)로 연결

즉, **실제 비밀값은 .env에 두고, yml에서는 그 값을 참조하는 구조**로 가는 것이 핵심이다.

---

## Spring 설정 파일 역할 정리
설정 파일은 역할을 분리해서 쓰는 것이 좋다.

1. **application.yaml**: 공통 설정 파일 (active profile, swagger, mybatis 공통 설정 등)
2. **application-dev.yml**: 개발 환경 전용 설정 파일 (datasource, JPA 개발 옵션, .env import 등)
3. **.env**: 실제 접속 정보 파일 (DB_URL, DB_USERNAME, DB_PASSWORD 등)

---

## application.yaml 작성 방법
```yaml
spring:
  profiles:
    active: dev

springdoc:
  swagger-ui:
    path: /swagger-ui.html
    operationsSorter: method

mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: com.pawcarechart.backend
  configuration:
    map-underscore-to-camel-case: true
```    
spring.profiles.active: dev: 기본 실행 환경을 dev로 지정하여 application-dev.yml이 함께 적용되도록 한다.

application-dev.yml 작성 방법
```YAML
spring:
  config:
    import: optional:file:./.env[.properties]

  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    open-in-view: false
spring.config.import: 현재 실행 경로 기준의 .env 파일을 읽는다.

spring.datasource: 실제 DB 연결 정보는 .env 변수를 참조한다.
```

## 🛠 .env 작성 방법
작성 예시:

```text
DB_URL=jdbc:postgresql://aws-1-ap-northeast-2.pooler.supabase.com:5432/postgres?sslmode=require
DB_USERNAME=postgres.<project-ref>
DB_PASSWORD=실제비밀번호
```

⚠️ 주의사항

값에 공백이나 따옴표를 넣지 않습니다.
실제 비밀번호를 yml 파일에 직접 노출하지 않고 .env를 통해 주입합니다.

## 📄 .gitignore 작성 방법
실제 .env 파일은 보안을 위해 Git에 업로드되지 않도록 설정해야 합니다.

설정 예시:
```text
.env
.env.*
!.env.example
```
## 💻 IntelliJ에서 확인할 부분
Run/Debug Configuration: 실행 대상이 올바른지 확인합니다.

Environment Variables: 이전에 설정해둔 환경 변수 값이 남아 있으면 현재 설정을 덮어쓸 수 있으므로 주의가 필요합니다.

Working Directory: .env 파일을 찾는 기준 경로(Root)가 맞는지 확인합니다.

## ✅ 정상 연결 확인 방법
애플리케이션 실행 로그에 아래와 같은 메시지가 출력되면 성공입니다.

HikariPool-1 - Added connection...

HikariPool-1 - Start completed.

Started Application

## 📌 요약 및 정리
Supabase는 PostgreSQL 기반 서비스입니다.

설정 파일은 application.yaml, application-dev.yml, .env로 역할을 나누어 관리하는 것이 권장됩니다.

IntelliJ의 Environment Variables와 Working Directory 설정 확인은 연결 오류 해결의 필수 단계입니다.