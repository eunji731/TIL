# 인프라 기본편 - Docker 실습 준비 문서
_환경변수 · 포트 매핑 · Compose · Docker Network · 배포 흐름 · 리눅스 기본 명령어_

> - Docker 기본 개념은 대충 이해했지만 아직 손에 안 익은 사람
> - `docker-compose.yml`을 봐도 무슨 뜻인지 잘 모르겠는 사람
> - 서버에 들어가서 뭘 쳐야 할지 막막한 사람
> - 실습하다가 길을 잃지 않도록, 순서와 이유까지 같이 알고 싶은 사람
>
---

## 목차

- [1. 문서 목적](#1-문서-목적)
- [2. 실습 전에 꼭 알아야 할 큰 그림](#2-실습-전에-꼭-알아야-할-큰-그림)
- [3. 환경변수란 무엇인가](#3-환경변수란-무엇인가)
  - [3.1 왜 application.yml에 다 안 쓰는가](#31-왜-applicationyml에-다-안-쓰는가)
  - [3.2 DB 주소/비밀번호는 어떻게 주입하는가](#32-db-주소비밀번호는-어떻게-주입하는가)
  - [3.3 Spring Boot에서 환경변수 읽는 예시](#33-spring-boot에서-환경변수-읽는-예시)
- [4. 포트란 무엇인가](#4-포트란-무엇인가)
  - [4.1 앱 포트 / 컨테이너 포트 / 서버 포트 차이](#41-앱-포트--컨테이너-포트--서버-포트-차이)
  - [4.2 `8080:8080`을 정확히 읽는 법](#42-80808080을-정확히-읽는-법)
  - [4.3 자주 보는 포트 예시](#43-자주-보는-포트-예시)
- [5. Compose 파일 구조 읽는 법](#5-compose-파일-구조-읽는-법)
  - [5.1 services](#51-services)
  - [5.2 ports](#52-ports)
  - [5.3 volumes](#53-volumes)
  - [5.4 environment](#54-environment)
  - [5.5 depends_on](#55-depends_on)
- [6. Docker Network란 무엇인가](#6-docker-network란-무엇인가)
  - [6.1 왜 컨테이너끼리는 localhost가 아닌 서비스명으로 붙는가](#61-왜-컨테이너끼리는-localhost가-아닌-서비스명으로-붙는가)
  - [6.2 자주 하는 실수](#62-자주-하는-실수)
- [7. 배포 흐름을 개발자 기준으로 이해하기](#7-배포-흐름을-개발자-기준으로-이해하기)
  - [7.1 코드 수정](#71-코드-수정)
  - [7.2 빌드](#72-빌드)
  - [7.3 이미지 생성](#73-이미지-생성)
  - [7.4 컨테이너 재기동](#74-컨테이너-재기동)
  - [7.5 로그 확인](#75-로그-확인)
- [8. 기본 리눅스 명령어](#8-기본-리눅스-명령어)
  - [8.1 ls](#81-ls)
  - [8.2 cd](#82-cd)
  - [8.3 pwd](#83-pwd)
  - [8.4 cat](#84-cat)
  - [8.5 less](#85-less)
  - [8.6 tail](#86-tail)
  - [8.7 grep](#87-grep)
  - [8.8 ps](#88-ps)
  - [8.9 ss 또는 netstat](#89-ss-또는-netstat)
  - [8.10 df -h](#810-df--h)
- [9. 실습에 바로 쓸 예제 구조](#9-실습에-바로-쓸-예제-구조)
- [10. 실습 1 - Spring + Postgres를 Compose로 띄워보기](#10-실습-1---spring--postgres를-compose로-띄워보기)
  - [10.1 준비 파일](#101-준비-파일)
  - [10.2 application.yml 작성](#102-applicationyml-작성)
  - [10.3 Dockerfile 작성](#103-dockerfile-작성)
  - [10.4 compose.yaml 작성](#104-composeyaml-작성)
  - [10.5 실행](#105-실행)
  - [10.6 확인](#106-확인)
- [11. 실습 2 - 일부러 흔히 하는 실수 내보기](#11-실습-2---일부러-흔히-하는-실수-내보기)
  - [11.1 DB_HOST를 localhost로 잘못 넣기](#111-db_host를-localhost로-잘못-넣기)
  - [11.2 포트 매핑 바꾸기](#112-포트-매핑-바꾸기)
  - [11.3 볼륨 없이 DB 띄우기](#113-볼륨-없이-db-띄우기)
- [12. 실습 중 막히면 보는 체크리스트](#12-실습-중-막히면-보는-체크리스트)
- [13. 자주 하는 질문](#13-자주-하는-질문)
- [14. 핵심 요약](#14-핵심-요약)

---

## 1. 문서 목적

- 환경변수가 왜 필요한지
- `8080:8080` 같은 포트 매핑을 어떻게 읽는지
- Compose 파일을 보고 구조를 해석할 수 있는지
- 컨테이너끼리는 왜 `localhost`가 아니라 서비스명으로 붙는지
- 코드 수정부터 컨테이너 재기동까지 전체 흐름이 어떤지
- 서버에서 기본 리눅스 명령어로 파일/로그/포트를 확인할 수 있는지

---

## 2. 실습 전에 꼭 알아야 할 큰 그림

실습은 보통 아래 흐름으로 진행됩니다.

```text
코드 작성/수정
  ↓
빌드
  ↓
Docker 이미지 생성
  ↓
컨테이너 실행
  ↓
로그 확인
  ↓
문제 수정 후 다시 빌드/실행
```

그리고 서비스 구조는 보통 아래처럼 됩니다.

```text
backend container
db container
```

또는

```text
frontend container
backend container
db container
nginx container
```

여기서 중요한 것은 다음입니다.

* 코드는 그냥 실행되지 않는다.
* 실행 환경이 필요하다.
* Docker는 그 실행 환경을 묶어준다.
* Compose는 여러 컨테이너를 같이 관리하게 해준다.

---

## 3. 환경변수란 무엇인가

환경변수는 한마디로 말하면:

> 프로그램이 실행될 때 바깥에서 주는 설정값


예를 들면 이런 값들을 환경변수로 많이 넣습니다.

* DB 주소
* DB 포트
* DB 이름
* DB 사용자명
* DB 비밀번호
* 실행 모드(dev/prod)
* API 키

---

### 3.1 왜 application.yml에 다 안 쓰는가

처음에는 이런 생각이 들 수 있습니다.

> 그냥 `application.yml`에 DB 주소랑 비밀번호 다 쓰면 되는 거 아닌가?

가능은 하지만, 실무에서는 보통 다 안 씁니다. 이유는 아래와 같습니다.

#### 1) 보안

비밀번호를 코드에 직접 적으면 Git에 올라갈 수 있습니다.

#### 2) 환경 분리

개발 환경, 테스트 환경, 운영 환경마다 값이 다를 수 있습니다.

예:

* 개발 DB 주소
* 운영 DB 주소

#### 3) 배포 편의성

코드는 그대로 두고, 실행할 때 환경변수만 바꿔서 다른 환경에 띄울 수 있습니다.

즉:

> 코드는 공통으로 두고, 달라지는 값은 환경변수로 뺀다

---

### 3.2 DB 주소/비밀번호는 어떻게 주입하는가

Compose에서는 보통 `environment`에 넣습니다.

예:

```yaml
services:
  backend:
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: mydb
      DB_USER: postgres
      DB_PASSWORD: 1234
```

이 뜻은:

* backend 컨테이너가 실행될 때
* `DB_HOST`, `DB_PORT` 같은 값을 같이 넘겨준다

즉 Spring 앱은 이 값을 받아서 DB에 연결합니다.

---

### 3.3 Spring Boot에서 환경변수 읽는 예시

#### `application.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
```

이 뜻은:

* `DB_HOST` 자리에 환경변수 값이 들어감
* `DB_PORT` 자리에 환경변수 값이 들어감
* `DB_NAME`, `DB_USER`, `DB_PASSWORD`도 마찬가지

즉 실제 실행할 때는 예를 들어 이렇게 해석됩니다.

```text
jdbc:postgresql://db:5432/mydb
```

---

## 4. 포트란 무엇인가

포트는 쉽게 말하면:

> 서버 안의 출입문 번호

입니다.

IP가 건물 주소라면, 포트는 몇 번 문으로 들어갈지 정하는 번호입니다.

---

### 4.1 앱 포트 / 컨테이너 포트 / 서버 포트 차이

이건 정말 중요합니다.

#### 앱 포트

앱이 실제로 내부에서 떠 있는 포트

예:

* Spring Boot가 8080에서 뜸

#### 컨테이너 포트

컨테이너 안에서 사용하는 포트

보통 앱 포트와 같을 수 있음

#### 서버 포트

외부에서 접속할 때 여는 포트

즉 경우에 따라 이렇게 될 수 있습니다.

```text
외부 9999 → 컨테이너 8080 → 앱 8080
```

---

### 4.2 `8080:8080`을 정확히 읽는 법

예:

```yaml
ports:
  - "8080:8080"
```

이건 이렇게 읽습니다.

> 서버(또는 내 PC) 8080 포트를
> 컨테이너 8080 포트에 연결한다

즉 왼쪽이 바깥, 오른쪽이 안쪽입니다.

#### 예시 1

```yaml
- "8080:8080"
```

```text
외부 8080 → 컨테이너 8080
```

#### 예시 2

```yaml
- "9999:8080"
```

```text
외부 9999 → 컨테이너 8080
```

즉 앱은 컨테이너 안에서 8080으로 떠 있는데,
사람은 바깥에서 9999로 접속할 수 있습니다.

---

### 4.3 자주 보는 포트 예시

* `80` = 일반 웹 접속
* `443` = HTTPS 접속
* `8080` = Spring Boot 등 개발/앱 포트
* `5432` = PostgreSQL
* `3000` = 프론트 개발 서버(자주 사용)

---

## 5. Compose 파일 구조 읽는 법

Compose 파일은 **서비스 구조 설명서**입니다.

예제:

```yaml
services:
  backend:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
      DB_PORT: 5432
    depends_on:
      - db

  db:
    image: postgres:15
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

---

### 5.1 services

`services`는 실행할 서비스 목록입니다.

예:

* `backend`
* `db`

즉 지금 이 시스템은 backend 컨테이너와 db 컨테이너를 같이 띄운다는 뜻입니다.

---

### 5.2 ports

`ports`는 바깥 포트와 컨테이너 포트를 연결합니다.

예:

```yaml
ports:
  - "8080:8080"
```

이건 외부 8080을 컨테이너 8080에 연결하는 뜻입니다.

---

### 5.3 volumes

`volumes`는 데이터를 남기기 위한 저장공간 연결입니다.

예:

```yaml
volumes:
  - pgdata:/var/lib/postgresql/data
```

이건 PostgreSQL 데이터가 Docker named volume `pgdata`에 저장된다는 뜻입니다.

---

### 5.4 environment

`environment`는 컨테이너 실행 시 주는 환경변수입니다.

예:

```yaml
environment:
  DB_HOST: db
  DB_PORT: 5432
```

즉 backend 컨테이너는 DB 주소를 `db`, 포트를 `5432`로 인식하게 됩니다.

---

### 5.5 depends_on

`depends_on`는 “이 서비스가 다른 서비스에 의존한다”는 뜻입니다.

예:

```yaml
depends_on:
  - db
```

즉 backend는 db가 필요하다는 뜻입니다.

주의할 점:

> `depends_on`은 순서 힌트일 뿐, DB가 완전히 정상 기동됨까지 보장하지는 않습니다.

---

## 6. Docker Network란 무엇인가

Docker Network는:

> 컨테이너끼리 서로 통신할 수 있게 해주는 내부 네트워크

입니다.

Compose로 띄운 컨테이너들은 보통 같은 네트워크에 묶입니다.

---

### 6.1 왜 컨테이너끼리는 localhost가 아닌 서비스명으로 붙는가

이건 꼭 이해해야 합니다.

#### 로컬 PC에서

`localhost` = 내 PC

#### 서버에서

`localhost` = 서버 자신

#### 컨테이너 안에서

`localhost` = 그 컨테이너 자기 자신

즉 backend 컨테이너 안에서 `localhost`는 backend 자신입니다.
DB가 아닙니다.

그래서 backend가 Postgres 컨테이너에 붙으려면 `localhost`가 아니라
Compose 서비스명인 `db`로 접근해야 합니다.

예:

```yaml
DB_HOST: db
```

그리고 datasource URL은:

```text
jdbc:postgresql://db:5432/mydb
```

가 되는 겁니다.

---

### 6.2 자주 하는 실수

#### 잘못된 예

```yaml
DB_HOST: localhost
```

이러면 backend 컨테이너는 자기 자신을 DB라고 착각하고 붙으려다가 실패할 수 있습니다.

#### 올바른 예

```yaml
DB_HOST: db
```

즉:

> 컨테이너끼리는 보통 `localhost`가 아니라 Compose 서비스명으로 붙는다

---

## 7. 배포 흐름을 개발자 기준으로 이해하기

### 7.1 코드 수정

먼저 코드나 설정을 바꿉니다.

예:

* Java 코드 수정
* `application.yml` 수정
* `Dockerfile` 수정
* `compose.yaml` 수정

---

### 7.2 빌드

Spring Boot라면 보통 jar를 다시 만듭니다.

예:

```bash
./gradlew build
```

또는 Maven이면:

```bash
mvn clean package
```

즉 코드 → 실행 파일 생성 단계입니다.

---

### 7.3 이미지 생성

```bash
docker build -t my-app .
```

이건 현재 Dockerfile 기준으로 이미지를 다시 만드는 작업입니다.

즉 빌드된 jar를 포함한 새 실행 원본을 만든다고 보면 됩니다.

---

### 7.4 컨테이너 재기동

단일 컨테이너면:

```bash
docker stop my-app
docker rm my-app
docker run -d -p 8080:8080 --name my-app my-app
```

Compose면 보통:

```bash
docker compose up -d --build
```

즉 새 이미지로 다시 올리는 과정입니다.

---

### 7.5 로그 확인

새로 올렸으면 바로 로그를 봅니다.

```bash
docker logs --tail 100 my-app
```

또는 Compose면:

```bash
docker compose logs --tail 100 backend
```

이 단계가 매우 중요합니다.

> 배포는 "띄웠다"에서 끝이 아니라
> "정상 동작하는지 확인했다"까지가 포함됩니다.

---

## 8. 기본 리눅스 명령어

실습할 때 아래 명령어는 정말 자주 씁니다.

---

### 8.1 ls

현재 폴더 안 파일/폴더 보기

```bash
ls
```

자세히 보기:

```bash
ls -al
```

---

### 8.2 cd

폴더 이동

```bash
cd 폴더명
```

예:

```bash
cd /home/ubuntu
```

상위 폴더로:

```bash
cd ..
```

---

### 8.3 pwd

현재 내가 어디 폴더에 있는지 확인

```bash
pwd
```

이건 실습 중 자주 헷갈릴 때 꼭 쓴다.

---

### 8.4 cat

파일 내용을 한 번에 출력

```bash
cat 파일명
```

예:

```bash
cat compose.yaml
```

짧은 파일 확인할 때 좋다.

---

### 8.5 less

긴 파일을 페이지 단위로 보기

```bash
less 파일명
```

예:

```bash
less backend.log
```

* 아래로 이동: 화살표
* 종료: `q`

---

### 8.6 tail

파일의 마지막 부분 보기

```bash
tail 파일명
```

마지막 100줄 보기:

```bash
tail -100 파일명
```

실시간으로 보기:

```bash
tail -f 파일명
```

로그 볼 때 가장 많이 쓴다.

---

### 8.7 grep

특정 단어 찾기

```bash
grep 찾을단어 파일명
```

예:

```bash
grep error backend.log
```

대소문자 무시:

```bash
grep -i error backend.log
```

앞뒤 문맥 같이 보기:

```bash
grep -i -C 3 error backend.log
```

---

### 8.8 ps

현재 실행 중인 프로세스 보기

```bash
ps -ef
```

자주 특정 프로세스 찾을 때 씀.

예:

```bash
ps -ef | grep java
```

---

### 8.9 ss 또는 netstat

어떤 포트가 열려 있는지 확인

```bash
ss -tulnp
```

또는 환경에 따라:

```bash
netstat -tulnp
```

예:

* 8080 포트가 열렸는지
* 5432가 열렸는지

확인할 때 유용하다.

---

### 8.10 df -h

디스크 사용량 보기

```bash
df -h
```

실습하다 보면 용량 부족도 문제가 될 수 있어서 알아두면 좋다.

---

## 9. 실습에 바로 쓸 예제 구조

이번 실습은 아주 단순한 구조로 진행합니다.

```text
backend
  ↓
postgres
```

즉:

* Spring Boot 앱 1개
* PostgreSQL 1개
* Compose로 같이 실행

이 구조만 해도 아래를 전부 체험할 수 있습니다.

* 환경변수
* 포트 매핑
* Compose
* Docker Network
* Volume
* 로그
* 재기동

---

## 10. 실습 1 - Spring + Postgres를 Compose로 띄워보기

---

### 10.1 준비 파일

프로젝트 루트에 아래 파일들이 있다고 가정합니다.

```text
project-root/
├─ Dockerfile
├─ compose.yaml
├─ build/libs/app.jar
└─ src/main/resources/application.yml
```

Spring Boot jar는 미리 빌드했다고 가정합니다.

---

### 10.2 application.yml 작성

```yaml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: update
```

#### 설명

* DB 주소를 코드에 박지 않고 환경변수로 받음
* `DB_HOST`에는 나중에 `db`가 들어갈 것
* `DB_PORT`에는 `5432`가 들어갈 것

---

### 10.3 Dockerfile 작성

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY build/libs/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 설명

* Java 17 실행 환경 사용
* jar 파일 복사
* 8080 포트 사용
* `java -jar app.jar`로 실행

---

### 10.4 compose.yaml 작성

```yaml
services:
  backend:
    build: .
    container_name: demo-backend
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: mydb
      DB_USER: postgres
      DB_PASSWORD: 1234
    depends_on:
      - db

  db:
    image: postgres:15
    container_name: demo-db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 1234
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

#### 핵심 포인트

* backend는 DB를 `localhost`가 아니라 `db`로 찾음
* db 데이터는 `pgdata` 볼륨에 저장
* backend와 db를 한 번에 띄움

---

### 10.5 실행

프로젝트 루트에서 실행합니다.

```bash
docker compose up -d --build
```

#### 뜻

* 필요하면 이미지 다시 빌드
* backend와 db 컨테이너 모두 백그라운드 실행

---

### 10.6 확인

#### 상태 확인

```bash
docker compose ps
```

#### backend 로그 확인

```bash
docker compose logs --tail 100 backend
```

#### db 로그 확인

```bash
docker compose logs --tail 100 db
```

#### 볼륨 확인

```bash
docker volume ls
```

#### 마운트 확인

```bash
docker inspect demo-db --format '{{json .Mounts}}'
```

---

## 11. 실습 2 - 일부러 흔히 하는 실수 내보기

실수도 일부러 해봐야 감이 옵니다.

---

### 11.1 DB_HOST를 localhost로 잘못 넣기

compose에서 일부러 이렇게 바꿔봅니다.

```yaml
environment:
  DB_HOST: localhost
```

그리고 다시 실행:

```bash
docker compose up -d --build
```

#### 예상 결과

backend가 DB 연결 실패할 가능성이 큽니다.

#### 왜냐면

backend 컨테이너 안에서 `localhost`는 backend 자신이지, db 컨테이너가 아니기 때문입니다.

#### 확인

```bash
docker compose logs --tail 100 backend
```

---

### 11.2 포트 매핑 바꾸기

예를 들어 backend를 이렇게 바꿔봅니다.

```yaml
ports:
  - "9999:8080"
```

다시 실행:

```bash
docker compose up -d --build
```

#### 뜻

외부는 9999로 들어와야 함
컨테이너 내부 앱은 여전히 8080

즉 접속 주소는 이제:

```text
http://localhost:9999
```

입니다.

---

### 11.3 볼륨 없이 DB 띄우기

db 쪽에서 `volumes`를 잠깐 빼고 띄워봅니다.

```yaml
db:
  image: postgres:15
  environment:
    POSTGRES_DB: mydb
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: 1234
```

이후 컨테이너를 내리고 다시 올려보면,
데이터가 유지되지 않거나 예기치 않은 문제가 생길 수 있습니다.

#### 여기서 배울 것

> DB 데이터는 볼륨이 있어야 안전하게 보존된다

---

## 12. 실습 중 막히면 보는 체크리스트

아래 순서로 보면 된다.

### 1) 현재 위치 확인

```bash
pwd
ls -al
```

### 2) Compose 파일 있는지 확인

```bash
cat compose.yaml
```

### 3) 컨테이너 상태 확인

```bash
docker compose ps
```

### 4) backend 로그 보기

```bash
docker compose logs --tail 100 backend
```

### 5) db 로그 보기

```bash
docker compose logs --tail 100 db
```

### 6) 포트 확인

```bash
ss -tulnp
```

### 7) 마운트 확인

```bash
docker inspect demo-db --format '{{json .Mounts}}'
```

### 8) 볼륨 확인

```bash
docker volume ls
```

---

## 13. 자주 하는 질문

### Q1. `depends_on` 있으면 DB가 완전히 준비될 때까지 기다리나?

아니요. 실행 순서 힌트일 뿐, DB가 완전히 ready 상태라는 뜻은 아닙니다.

### Q2. 왜 DB_HOST를 localhost로 하면 안 되나?

컨테이너 안에서 localhost는 자기 자신이기 때문입니다. 다른 컨테이너는 서비스명으로 접근해야 합니다.

### Q3. 포트는 왜 두 개를 쓰나?

바깥에서 들어오는 문과 컨테이너 안 프로그램 문을 연결하기 때문입니다.

### Q4. DB 데이터는 어디에 남나?

볼륨을 썼다면 Docker volume 또는 마운트된 서버 경로에 남습니다.

### Q5. 로그가 너무 길면 어떻게 하나?

`--tail`, `grep`, `less`, `tail -f`를 같이 사용합니다.

---

## 14. 핵심 요약

* **환경변수**는 바깥에서 주는 설정값이다.
* **application.yml에 다 박아두지 않는 이유**는 보안과 환경 분리 때문이다.
* **포트 매핑**은 바깥 포트와 컨테이너 포트를 연결하는 것이다.
* `8080:8080`은 **외부 8080 → 컨테이너 8080**이다.
* **Compose**는 여러 컨테이너 구조를 한 번에 정의하고 실행한다.
* `services`, `ports`, `volumes`, `environment`, `depends_on`는 꼭 읽을 수 있어야 한다.
* **컨테이너끼리는 localhost가 아니라 서비스명으로 붙는 경우가 많다.**
* **배포 흐름**은 코드 수정 → 빌드 → 이미지 생성 → 컨테이너 재기동 → 로그 확인이다.
* `ls`, `cd`, `pwd`, `cat`, `less`, `tail`, `grep`, `ps`, `ss`, `df -h`는 꼭!