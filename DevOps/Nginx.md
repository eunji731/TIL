````md
# Nginx / Docker Compose / Spring 로그 운영 가이드

> 이 문서는 Nginx, Docker Compose, Spring Boot 로그를 처음 접하는 개발자도 운영 구조를 이해하고 장애를 점검할 수 있도록 정리한 문서이다.  
> 특히 `Nginx → Spring → DB` 구조와 Docker 기반 배포 환경에서 자주 발생하는 이슈를 빠르게 파악하는 것을 목표로 한다.

---

## 목차

- [1. 문서 개요](#1-문서-개요)
- [2. 전체 아키텍처 이해](#2-전체-아키텍처-이해)
  - [2.1 기본 구조](#21-기본-구조)
  - [2.2 요청 흐름](#22-요청-흐름)
- [3. Nginx 개념 정리](#3-nginx-개념-정리)
  - [3.1 Nginx란](#31-nginx란)
  - [3.2 Nginx를 사용하는 목적](#32-nginx를-사용하는-목적)
  - [3.3 Nginx의 주요 역할](#33-nginx의-주요-역할)
- [4. Nginx + Spring 실제 배포 구조](#4-nginx--spring-실제-배포-구조)
  - [4.1 기본 배포 구조](#41-기본-배포-구조)
  - [4.2 왜 Spring 앞에 Nginx를 두는가](#42-왜-spring-앞에-nginx를-두는가)
- [5. Docker + Nginx 구조](#5-docker--nginx-구조)
  - [5.1 Docker 환경의 기본 구조](#51-docker-환경의-기본-구조)
  - [5.2 Docker에서 주의할 점](#52-docker에서-주의할-점)
- [6. docker-compose.yml 읽는 방법](#6-docker-composeyml-읽는-방법)
  - [6.1 services](#61-services)
  - [6.2 ports](#62-ports)
  - [6.3 volumes](#63-volumes)
  - [6.4 depends_on](#64-depends_on)
  - [6.5 environment](#65-environment)
- [7. Docker Compose 주요 명령어](#7-docker-compose-주요-명령어)
- [8. Nginx 설정 파일 읽는 방법](#8-nginx-설정-파일-읽는-방법)
  - [8.1 server](#81-server)
  - [8.2 listen](#82-listen)
  - [8.3 server_name](#83-server_name)
  - [8.4 location](#84-location)
  - [8.5 proxy_pass](#85-proxy_pass)
  - [8.6 proxy_set_header](#86-proxy_set_header)
- [9. Nginx 로그 읽는 방법](#9-nginx-로그-읽는-방법)
  - [9.1 access log](#91-access-log)
  - [9.2 error log](#92-error-log)
- [10. Spring 로그 읽는 방법](#10-spring-로그-읽는-방법)
  - [10.1 정상 기동 로그](#101-정상-기동-로그)
  - [10.2 자주 발생하는 오류](#102-자주-발생하는-오류)
  - [10.3 로그 확인 시 핵심 포인트](#103-로그-확인-시-핵심-포인트)
- [11. 502 Bad Gateway 이해](#11-502-bad-gateway-이해)
  - [11.1 의미](#111-의미)
  - [11.2 주요 원인](#112-주요-원인)
- [12. 장애 발생 시 확인 순서](#12-장애-발생-시-확인-순서)
- [13. 실무 점검 체크리스트](#13-실무-점검-체크리스트)
- [14. 예시 파일](#14-예시-파일)
  - [14.1 docker-compose.yml](#141-docker-composeyml)
  - [14.2 default.conf](#142-defaultconf)
- [15. 자주 쓰는 명령어 모음](#15-자주-쓰는-명령어-모음)
- [16. 핵심 요약](#16-핵심-요약)
- [17. 결론](#17-결론)

---

## 1. 문서 개요

본 문서는 아래 항목을 한 번에 이해할 수 있도록 정리한 운영 문서이다.

- Nginx의 개념과 사용 목적
- Nginx + Spring 실제 배포 구조
- Docker + Nginx 구조
- `docker-compose.yml` 읽는 방법
- `nginx.conf` 또는 `default.conf` 읽는 방법
- Nginx 로그와 Spring 로그를 읽는 방법
- `502 Bad Gateway` 원인과 해석
- 장애 발생 시 어디부터 확인해야 하는지에 대한 운영 절차

이 문서는 다음과 같은 상황에서 활용할 수 있다.

- 배포 구조를 처음 이해해야 하는 경우
- 운영 서버 장애 원인을 파악해야 하는 경우
- Nginx와 Spring 사이 연결 문제를 확인해야 하는 경우
- Docker Compose 기반 시스템을 유지보수해야 하는 경우

---

## 2. 전체 아키텍처 이해

### 2.1 기본 구조

일반적으로 웹 서비스는 다음과 같은 구조로 운영된다.

```text
사용자 브라우저
   ↓
Nginx
   ↓
Spring Boot
   ↓
DB
````

Docker 환경에서는 보통 아래와 같이 구성된다.

```text
사용자 브라우저
   ↓
Nginx 컨테이너
   ↓
Spring 컨테이너
   ↓
DB 컨테이너
```

즉, 사용자는 보통 Spring Boot 애플리케이션에 직접 접속하지 않고, Nginx를 통해 접속하게 된다.

---

### 2.2 요청 흐름

사용자가 다음 URL로 접속한다고 가정한다.

```text
https://example.com/api/boards
```

실제 내부 요청 흐름은 아래와 같다.

```text
브라우저
   ↓
Nginx
   ↓
Spring Controller
   ↓
Service
   ↓
Repository / Mapper
   ↓
DB
```

흐름을 순서대로 설명하면 다음과 같다.

1. 사용자가 브라우저에서 URL을 호출한다.
2. 요청은 먼저 Nginx가 받는다.
3. Nginx는 설정에 따라 요청을 Spring Boot 서버로 전달한다.
4. Spring 애플리케이션이 비즈니스 로직을 수행한다.
5. 필요한 경우 DB 조회 또는 저장을 수행한다.
6. 결과를 다시 사용자에게 응답한다.

---

## 3. Nginx 개념 정리

### 3.1 Nginx란

Nginx는 웹 서버이면서 동시에 리버스 프록시(Reverse Proxy) 역할을 하는 소프트웨어이다.

쉽게 말하면 다음과 같다.

> 외부 요청을 가장 먼저 받아서 내부 서버로 적절히 전달해주는 앞단 서버

즉, 클라이언트는 Nginx와 통신하고, Nginx가 실제 애플리케이션 서버(Spring 등)와 통신한다.

---

### 3.2 Nginx를 사용하는 목적

Spring Boot 애플리케이션만으로도 서비스는 가능하다.
예를 들어 `8080` 포트로 직접 실행할 수 있다.

하지만 실무에서는 보통 Spring 앞단에 Nginx를 둔다. 주요 이유는 아래와 같다.

1. 외부 요청을 받아 내부 서버로 전달하기 위해
2. HTTPS(SSL 인증서) 처리를 맡기기 위해
3. CSS, JS, 이미지 같은 정적 파일을 효율적으로 처리하기 위해
4. 접근 제어, 요청 제한, 업로드 크기 제한 등을 적용하기 위해
5. 외부와 내부 서버 구조를 분리하여 운영하기 위해

---

### 3.3 Nginx의 주요 역할

#### 1) Reverse Proxy

가장 대표적인 역할이다.

```text
사용자 → Nginx → Spring
```

사용자가 Spring에 직접 붙는 것이 아니라 Nginx가 대신 요청을 받고 전달한다.

---

#### 2) HTTPS 처리

보통 SSL 인증서 적용은 Nginx에서 한다.

```text
사용자(HTTPS)
   ↓
Nginx(SSL 처리)
   ↓
Spring(HTTP)
```

즉, 외부는 HTTPS로 접속하되 내부 서버는 HTTP로 운영하는 구조가 가능하다.

---

#### 3) 정적 파일 처리

이미지, CSS, JS 파일은 Spring이 처리할 필요 없이 Nginx가 직접 응답할 수 있다.
이렇게 하면 Spring은 API와 비즈니스 로직 처리에 더 집중할 수 있다.

---

#### 4) 보안 및 제어

Nginx에서는 다음과 같은 작업이 가능하다.

* 특정 IP 차단
* 업로드 용량 제한
* 요청 제한(rate limit)
* 특정 URL 경로 제어

---

#### 5) 로드밸런싱

서버가 여러 대일 경우 요청을 분산시킬 수 있다.

```text
사용자
   ↓
Nginx
  ├─ Spring 서버 1
  ├─ Spring 서버 2
  └─ Spring 서버 3
```

---

## 4. Nginx + Spring 실제 배포 구조

### 4.1 기본 배포 구조

Docker를 쓰지 않는 가장 기본적인 구조는 아래와 같다.

```text
인터넷
   ↓
도메인(example.com)
   ↓
Nginx (80 / 443)
   ↓
Spring Boot (8080)
   ↓
PostgreSQL (5432)
```

각 요소의 역할은 다음과 같다.

* **Nginx**: 외부 요청을 받는 웹 서버
* **Spring Boot**: 실제 비즈니스 로직을 처리하는 애플리케이션 서버
* **PostgreSQL**: 데이터를 저장하는 데이터베이스

---

### 4.2 왜 Spring 앞에 Nginx를 두는가

#### 1) 포트 분리

사용자는 보통 `80`, `443` 포트로 접속한다.
반면 Spring Boot는 보통 `8080` 같은 내부 포트를 사용한다.

예:

```text
사용자: https://example.com
          ↓
Nginx: 443 수신
          ↓
Spring: localhost:8080
```

---

#### 2) HTTPS 적용이 쉬움

SSL 인증서와 HTTPS 처리를 Spring이 아닌 Nginx가 맡음으로써 설정이 단순해진다.

---

#### 3) 운영 구조가 깔끔해짐

외부에 Spring 포트를 직접 노출하지 않아도 되므로 보안과 구조 분리 측면에서 유리하다.

---

## 5. Docker + Nginx 구조

### 5.1 Docker 환경의 기본 구조

Docker를 사용하는 경우 보통 각 구성요소를 각각의 컨테이너로 분리한다.

```text
[Docker Host]
 ├─ nginx container
 ├─ backend container
 └─ db container
```

요청 흐름은 다음과 같다.

```text
사용자
   ↓
호스트 80/443
   ↓
nginx container
   ↓
backend container:8080
   ↓
db container:5432
```

---

### 5.2 Docker에서 주의할 점

Docker에서는 `localhost` 해석을 조심해야 한다.

> 컨테이너 안에서 `localhost`는 자기 자신을 의미한다.

즉, Nginx 컨테이너 안에서 아래 설정은 위험할 수 있다.

```nginx
proxy_pass http://localhost:8080;
```

이 경우 Nginx 컨테이너 내부의 자기 자신을 가리킬 수 있다.

Docker Compose에서는 보통 서비스명으로 연결해야 한다.

```nginx
proxy_pass http://backend:8080;
```

여기서 `backend`는 `docker-compose.yml`에 정의된 서비스명이다.

---

## 6. docker-compose.yml 읽는 방법

Docker Compose 파일은 전체 컨테이너 구조를 정의하는 설계도라고 보면 된다.

예시:

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - backend

  backend:
    image: my-spring-app
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
```

---

### 6.1 services

```yaml
services:
  nginx:
  backend:
  db:
```

실행할 서비스 목록이다.

* `nginx`: 웹 서버 및 프록시
* `backend`: Spring Boot 애플리케이션
* `db`: 데이터베이스

---

### 6.2 ports

```yaml
ports:
  - "80:80"
```

형식은 다음과 같다.

```text
호스트포트:컨테이너포트
```

예:

```yaml
ports:
  - "80:80"
```

의미:

```text
서버의 80 포트 → nginx 컨테이너의 80 포트
```

예:

```yaml
ports:
  - "8080:8080"
```

의미:

```text
서버의 8080 포트 → backend 컨테이너의 8080 포트
```

---

### 6.3 volumes

```yaml
volumes:
  - ./default.conf:/etc/nginx/conf.d/default.conf
```

의미는 다음과 같다.

* 서버에 있는 `./default.conf` 파일을
* 컨테이너 내부 `/etc/nginx/conf.d/default.conf` 파일로 연결한다.

즉, 서버에서 설정 파일을 수정하면 컨테이너에서 해당 파일을 사용할 수 있다.

---

### 6.4 depends_on

```yaml
depends_on:
  - backend
```

의미:

* `nginx` 서비스가 `backend` 서비스와 의존 관계를 가진다는 뜻이다.

주의할 점은 다음과 같다.

> `depends_on`은 실행 순서 힌트일 뿐, backend가 완전히 정상 기동되었음을 보장하지는 않는다.

즉, `backend` 컨테이너가 떠 있어도 Spring 내부 오류가 있으면 Nginx 연결은 실패할 수 있다.

---

### 6.5 environment

```yaml
environment:
  - SPRING_PROFILES_ACTIVE=prod
```

애플리케이션 실행에 필요한 환경변수를 전달한다.

예를 들면 다음이 들어갈 수 있다.

* Spring profile
* DB URL
* DB 계정/비밀번호
* API Key
* 기타 운영 설정값

---

## 7. Docker Compose 주요 명령어

### 컨테이너 상태 확인

```bash
docker compose ps
```

현재 서비스가 살아 있는지 가장 먼저 확인할 때 사용한다.

---

### 전체 서비스 실행

```bash
docker compose up -d
```

백그라운드로 컨테이너를 띄운다.

---

### 이미지 재빌드 포함 실행

```bash
docker compose up -d --build
```

소스나 Dockerfile이 변경된 뒤 다시 띄울 때 사용한다.

---

### 서비스 중지

```bash
docker compose down
```

컨테이너를 중지하고 내린다.

---

### 특정 서비스 재시작

```bash
docker compose restart nginx
docker compose restart backend
```

---

### 전체 재시작

```bash
docker compose restart
```

---

## 8. Nginx 설정 파일 읽는 방법

Nginx 설정 파일은 보통 `nginx.conf` 또는 `default.conf`를 사용한다.

대표 예시는 다음과 같다.

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

### 8.1 server

```nginx
server {
    ...
}
```

하나의 서버 설정 단위이다.
어떤 포트, 어떤 도메인, 어떤 요청을 어떻게 처리할지 정의한다.

---

### 8.2 listen

```nginx
listen 80;
```

Nginx가 어떤 포트를 수신할지를 정의한다.

예:

* `80`: HTTP
* `443`: HTTPS

---

### 8.3 server_name

```nginx
server_name example.com;
```

어떤 도메인 요청을 이 설정이 처리할지 지정한다.

---

### 8.4 location

```nginx
location / {
    ...
}
```

어떤 URL 경로를 어떤 방식으로 처리할지 정의한다.

예:

* `location /` → 전체 요청
* `location /api/` → `/api/` 요청
* `location /static/` → 정적 파일 요청

---

### 8.5 proxy_pass

```nginx
proxy_pass http://backend:8080;
```

가장 중요한 설정이다.

의미:

> 현재 요청을 `backend:8080`으로 전달한다.

즉, Nginx가 직접 응답하지 않고 Spring 애플리케이션으로 프록시한다는 뜻이다.

---

### 8.6 proxy_set_header

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

프록시 뒤에 있는 Spring 서버가 원래 요청 정보를 알 수 있도록 헤더를 전달하는 설정이다.

주요 역할은 다음과 같다.

* 원래 Host 전달
* 실제 사용자 IP 전달
* 프록시 체인 정보 전달

운영 환경에서는 거의 기본적으로 사용하는 설정이다.

---

## 9. Nginx 로그 읽는 방법

Nginx 로그는 크게 두 가지를 본다.

* Access Log
* Error Log

로그 위치는 환경에 따라 다를 수 있지만 일반적으로 아래와 같다.

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Docker에서는 보통 아래 명령으로 확인한다.

```bash
docker compose logs nginx
docker compose logs -f nginx
```

---

### 9.1 access log

사용자가 어떤 요청을 보냈고 어떤 응답 코드가 반환되었는지를 기록한다.

예시:

```text
GET /api/users 200
GET /api/login 500
GET /favicon.ico 404
```

주요 상태코드는 다음과 같다.

* `200`: 정상
* `404`: 경로 없음
* `500`: 백엔드 서버 내부 오류
* `502`: 프록시 대상 연결 실패
* `504`: 응답 시간 초과

---

### 9.2 error log

실제 장애 원인을 파악할 때 더 중요하다.

예시:

```text
connect() failed (111: Connection refused) while connecting to upstream
```

의미:

> Nginx가 upstream(뒤쪽 서버, 보통 Spring)에 연결하려 했지만 실패했다.

이 경우 보통 아래 원인을 의심한다.

* backend 컨테이너가 죽어 있음
* backend 포트가 틀림
* 서비스명이 틀림
* 네트워크 문제
* Spring이 아직 완전히 기동되지 않음

또 다른 예시는 다음과 같다.

```text
host not found in upstream "backend"
```

의미:

* `backend`라는 서비스명을 Nginx가 찾지 못함
* Compose 서비스명 오타 또는 네트워크 문제일 가능성이 높음

---

## 10. Spring 로그 읽는 방법

Spring 로그는 애플리케이션이 왜 안 뜨는지 확인하는 핵심 자료이다.

Docker 환경에서는 보통 아래 명령으로 확인한다.

```bash
docker compose logs backend
docker compose logs -f backend
```

---

### 10.1 정상 기동 로그

정상적으로 떴을 때 자주 보이는 로그 예시는 다음과 같다.

```text
Tomcat started on port(s): 8080 (http)
Started Application in 12.345 seconds
```

이 메시지가 있으면 기본적으로 Spring 애플리케이션은 정상 기동된 상태로 볼 수 있다.

---

### 10.2 자주 발생하는 오류

#### 1) DB 연결 실패

```text
Failed to connect to database
```

의심 포인트:

* DB 컨테이너가 꺼져 있음
* DB 주소/포트가 틀림
* 계정 정보가 틀림
* 네트워크 연결 문제

---

#### 2) 포트 충돌

```text
Port 8080 is already in use
```

의미:

* 다른 프로세스가 이미 해당 포트를 점유 중

---

#### 3) 환경변수 누락

```text
Could not resolve placeholder 'XXX'
```

의미:

* `application.yml`
* 환경변수
* Docker Compose 설정

중 하나에 필수값이 빠졌을 가능성이 높다.

---

#### 4) Bean 생성 실패

```text
Error creating bean with name '...'
```

의미:

* Spring 설정 문제
* 의존성 주입 문제
* 잘못된 설정 클래스 또는 구성 오류

---

#### 5) 실제 원인 확인 포인트

Java 예외는 보통 맨 아래쪽 `Caused by:` 부분이 핵심이다.

예외 메시지가 길더라도 다음을 중점적으로 본다.

* `Exception`
* `Caused by`
* 최초 발생 원인 메시지

---

### 10.3 로그 확인 시 핵심 포인트

Spring 로그를 읽을 때는 다음 순서로 본다.

1. 애플리케이션이 정상 시작되었는가
2. Tomcat이 포트를 정상적으로 열었는가
3. DB 연결이 성공했는가
4. 예외가 발생했는가
5. `Caused by:` 아래 실제 원인이 무엇인가

---

## 11. 502 Bad Gateway 이해

### 11.1 의미

`502 Bad Gateway`는 운영 중 가장 자주 마주치는 에러 중 하나이다.

의미는 다음과 같다.

> Nginx는 살아 있지만, 뒤쪽 백엔드(Spring)와 정상적으로 통신하지 못하는 상태

즉, 화면에 502가 보인다고 해서 무조건 Nginx 자체가 문제인 것은 아니다.
오히려 뒤에 있는 Spring 또는 연결 경로 문제일 가능성이 더 높다.

---

### 11.2 주요 원인

#### 1) backend 컨테이너가 죽어 있음

가장 흔한 원인이다.

---

#### 2) `proxy_pass` 대상이 틀림

예:

```nginx
proxy_pass http://backend:8080;
```

그런데 실제 서비스명은 `app`일 수 있다.

---

#### 3) 포트가 틀림

Spring은 `8081`로 떠 있는데 Nginx는 `8080`으로 보내는 경우

---

#### 4) Docker 네트워크 문제

같은 네트워크에 묶이지 않았거나, 서비스 이름 해석이 실패한 경우

---

#### 5) Spring이 아직 기동 중

Nginx가 먼저 요청을 보내는데 Spring이 아직 준비 중이면 일시적으로 502가 발생할 수 있다.

---

## 12. 장애 발생 시 확인 순서

서비스 장애가 발생했을 때는 무작정 재시작하지 말고, 아래 순서로 점검하는 것이 좋다.

### 1단계. 컨테이너 상태 확인

```bash
docker compose ps
```

확인 포인트:

* `nginx`가 `running` 상태인가
* `backend`가 `running` 상태인가
* `db`가 `running` 상태인가

특히 `backend`가 `exited` 상태이면 502가 발생할 가능성이 높다.

---

### 2단계. backend 로그 확인

```bash
docker compose logs backend
```

확인 포인트:

* Spring이 정상 기동했는가
* DB 연결 실패는 없는가
* 환경변수 누락은 없는가
* 예외가 발생했는가

---

### 3단계. nginx 로그 확인

```bash
docker compose logs nginx
```

확인 포인트:

* `connection refused`
* `host not found`
* `upstream` 관련 오류
* 설정 파일 문법 오류

---

### 4단계. Nginx 설정 확인

특히 아래 부분을 본다.

```nginx
proxy_pass http://backend:8080;
```

확인 포인트:

* 서비스명이 실제 Compose 서비스명과 같은가
* 포트가 맞는가
* 경로(location)가 올바른가

---

### 5단계. DB 상태 확인

```bash
docker compose logs db
```

DB 문제로 Spring이 기동 실패할 수 있으므로 함께 확인한다.

---

## 13. 실무 점검 체크리스트

아래 체크리스트는 운영 장애 대응 시 빠르게 확인하기 위한 용도이다.

### 기본 상태 점검

* [ ] `docker compose ps` 실행
* [ ] `nginx` 상태 확인
* [ ] `backend` 상태 확인
* [ ] `db` 상태 확인

### backend 점검

* [ ] Spring 정상 기동 로그 확인
* [ ] DB 연결 성공 여부 확인
* [ ] `Caused by` 하위 예외 확인
* [ ] 환경변수 누락 여부 확인

### nginx 점검

* [ ] `docker compose logs nginx` 확인
* [ ] `upstream` 관련 에러 확인
* [ ] `connection refused` 확인
* [ ] `host not found` 확인
* [ ] `proxy_pass` 대상 확인

### 설정 점검

* [ ] Compose 서비스명과 Nginx 서비스명이 일치하는가
* [ ] 포트가 맞는가
* [ ] 같은 네트워크에 있는가
* [ ] 설정 파일 수정 후 재시작/재적용했는가

---

## 14. 예시 파일

### 14.1 docker-compose.yml

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - backend

  backend:
    image: my-spring-app
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
```

---

### 14.2 default.conf

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## 15. 자주 쓰는 명령어 모음

### 상태 확인

```bash
docker compose ps
```

### 전체 실행

```bash
docker compose up -d
```

### 재빌드 포함 실행

```bash
docker compose up -d --build
```

### 전체 중지

```bash
docker compose down
```

### 전체 로그 확인

```bash
docker compose logs
```

### nginx 로그 확인

```bash
docker compose logs nginx
docker compose logs -f nginx
```

### backend 로그 확인

```bash
docker compose logs backend
docker compose logs -f backend
```

### db 로그 확인

```bash
docker compose logs db
```

### 전체 재시작

```bash
docker compose restart
```

### 특정 서비스 재시작

```bash
docker compose restart nginx
docker compose restart backend
docker compose restart db
```

---

## 16. 핵심 요약

### 1) 사용자는 보통 Spring에 직접 붙지 않는다

대부분 `Nginx → Spring → DB` 구조를 사용한다.

---

### 2) Nginx는 문지기이자 전달자이다

Nginx는 외부 요청을 가장 먼저 받고, 내부 Spring 서버로 전달한다.

---

### 3) Docker에서는 `localhost`를 조심해야 한다

컨테이너 내부의 `localhost`는 자기 자신이다.
컨테이너 간 연결은 보통 서비스명으로 한다.

예:

```nginx
proxy_pass http://backend:8080;
```

---

### 4) 502는 대부분 backend 또는 연결 문제이다

화면에 502가 떠도 실제 원인은 아래일 수 있다.

* backend가 죽음
* 포트 불일치
* 서비스명 오타
* 네트워크 문제
* Spring 기동 실패

---

### 5) 장애 대응은 순서가 중요하다

추천 점검 순서:

1. `docker compose ps`
2. `docker compose logs backend`
3. `docker compose logs nginx`
4. `proxy_pass` 확인
5. DB 상태 확인

---

## 17. 결론

이 문서의 핵심 목적은 단순히 Nginx나 Docker Compose 명령어를 외우는 것이 아니다.

운영 중 문제가 생겼을 때 다음을 판단할 수 있도록 하는 것이 목표이다.

* 지금 문제는 Nginx 문제인가
* Spring 문제인가
* Docker Compose 문제인가
* DB 문제인가
* 아니면 서비스 간 연결 문제인가

즉, 화면에 에러가 보였을 때 단순히 “서버가 안 된다”에서 멈추는 것이 아니라,
어느 레이어에서 문제가 발생했는지 구분하고 점검 순서를 세울 수 있어야 한다.

이 문서를 기준으로 기본 구조와 로그 확인 방법을 익혀두면,
향후 Docker 기반 Spring 서비스 운영과 장애 대응에 필요한 최소한의 기준점을 잡을 수 있다.

---

## 부록. 추천 파일 구조

```text
project-root/
├─ README.md
├─ docker-compose.yml
└─ nginx/
   └─ default.conf
```
