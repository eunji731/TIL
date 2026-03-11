# 회사 인프라 기본편 - 개발자용 Docker / Volume / 실행 구조 문서

> **이미지 / 컨테이너 / 볼륨 / 네트워크 / 명령어 / 로그 확인 / 현재 서버 구조 해석
>
> - Docker의 개발자 관점 개념
> - Docker를 왜 쓰는지
> - Docker 핵심 구성요소
> - Docker 명령어 세트와 해석
> - Volume이 무엇인지
> - Volume을 왜 쓰는지
> - OCI 서버에서 Volume을 어떻게 이해해야 하는지
> - 현재 운영 서버에서 실제로 어떤 볼륨을 쓰고 있는지
> - 장애 확인 시 어디를 봐야 하는지

---

## 목차

- [1. 문서 목적](#1-문서-목적)
- [2. Docker를 개발자 수준으로 이해하기](#2-docker를-개발자-수준으로-이해하기)
- [3. Docker를 왜 사용하는가](#3-docker를-왜-사용하는가)
- [4. Docker 핵심 개념](#4-docker-핵심-개념)
  - [4.1 Image](#41-image)
  - [4.2 Container](#42-container)
  - [4.3 Dockerfile](#43-dockerfile)
  - [4.4 Docker Compose](#44-docker-compose)
  - [4.5 Volume](#45-volume)
  - [4.6 Docker Network](#46-docker-network)
- [5. Docker를 비유로 이해하기](#5-docker를-비유로-이해하기)
- [6. Docker 실제 사용 흐름](#6-docker-실제-사용-흐름)
- [7. 명령어 세트와 해석](#7-명령어-세트와-해석)
  - [7.1 이미지 빌드](#71-이미지-빌드)
  - [7.2 컨테이너 실행](#72-컨테이너-실행)
  - [7.3 실행 중인 컨테이너 확인](#73-실행-중인-컨테이너-확인)
  - [7.4 전체 컨테이너 확인](#74-전체-컨테이너-확인)
  - [7.5 이미지 목록 확인](#75-이미지-목록-확인)
  - [7.6 컨테이너 로그 보기](#76-컨테이너-로그-보기)
  - [7.7 컨테이너 내부 접속](#77-컨테이너-내부-접속)
  - [7.8 컨테이너 중지 및 삭제](#78-컨테이너-중지-및-삭제)
  - [7.9 Compose 전체 실행](#79-compose-전체-실행)
  - [7.10 Compose 상태 확인](#710-compose-상태-확인)
  - [7.11 Compose 로그 확인](#711-compose-로그-확인)
  - [7.12 Compose 전체 종료](#712-compose-전체-종료)
- [8. Volume이란 무엇인가](#8-volume이란-무엇인가)
- [9. Volume을 왜 사용하는가](#9-volume을-왜-사용하는가)
- [10. Volume을 안 쓰면 생길 수 있는 문제](#10-volume을-안-쓰면-생길-수-있는-문제)
- [11. Volume 사용 방식 2가지](#11-volume-사용-방식-2가지)
  - [11.1 Docker Named Volume](#111-docker-named-volume)
  - [11.2 Bind Mount](#112-bind-mount)
- [12. Docker에서 Volume 사용하는 예시](#12-docker에서-volume-사용하는-예시)
- [13. Compose에서 Volume 사용하는 예시](#13-compose에서-volume-사용하는-예시)
- [14. OCI에서 Volume을 어떻게 이해해야 하는가](#14-oci에서-volume을-어떻게-이해해야-하는가)
- [15. Docker Volume과 OCI Volume의 차이](#15-docker-volume과-oci-volume의-차이)
- [16. 현재 운영 서버 구조 확인 결과](#16-현재-운영-서버-구조-확인-결과)
  - [16.1 현재 컨테이너 목록](#161-현재-컨테이너-목록)
  - [16.2 Postgres 컨테이너 Volume 확인 결과](#162-postgres-컨테이너-volume-확인-결과)
  - [16.3 현재 DB 저장 구조 해석](#163-현재-db-저장-구조-해석)
- [17. unhealthy 상태란 무엇인가](#17-unhealthy-상태란-무엇인가)
- [18. 로그가 너무 많을 때 확인하는 방법](#18-로그가-너무-많을-때-확인하는-방법)
- [19. 현재 서버에서 주로 확인할 명령어](#19-현재-서버에서-주로-확인할-명령어)
- [20. 개발자가 꼭 알아야 할 핵심 정리](#20-개발자가-꼭-알아야-할-핵심-정리)
- [21. 결론](#21-결론)

---

## 1. 문서 목적

**개발자 수준에서 이해하기 위한 기본 정리 문서**

- 이미지와 컨테이너는 어떻게 다른가?
- Compose 파일은 어떤 역할을 하는가?
- Volume은 왜 필요한가?
- DB 데이터는 어디에 저장되는가?
- 현재 운영 서버에서는 어떤 구조로 돌아가고 있는가?
- 문제가 났을 때 어떤 명령어를 어디에 써야 하는가?

---

## 2. Docker를 개발자 수준으로 이해하기
> 프로그램을 환경째 포장해서 실행하는 도구
> Docker는 애플리케이션을 **일관된 환경에서**, **격리된 형태로**, **재현 가능하게** 실행하기 위한 컨테이너 기반 플랫폼입니다.

개발자 입장에서 중요한 키워드는 아래 3개입니다.

- **일관성**: 로컬 / 테스트 / 운영에서 비슷하게 실행
- **격리**: 프로그램끼리 덜 꼬이게 실행
- **재현성**: 누가 어디서 실행해도 비슷한 결과가 나옴

---

## 3. Docker를 왜 사용하는가

Docker를 사용하는 주요 이유는 다음과 같습니다.

### 1) 환경 차이를 줄이기 위해
같은 프로그램이 로컬에서는 되는데 서버에서는 안 되는 문제를 줄이기 위해 사용합니다.

### 2) 실행 환경을 같이 묶기 위해
프로그램만이 아니라, 해당 프로그램이 실행되기 위한 환경까지 함께 묶을 수 있습니다.

### 3) 배포를 쉽게 하기 위해
서버에 직접 이것저것 설치하는 대신, 준비된 이미지로 빠르게 실행할 수 있습니다.

### 4) 프로그램 간 충돌을 줄이기 위해
자바 버전, 라이브러리 버전, 설정 차이 등으로 인해 프로그램끼리 꼬이는 문제를 줄일 수 있습니다.

### 5) 서비스 구조를 나눠서 관리하기 위해
예를 들면 아래처럼 서비스별로 분리할 수 있습니다.

```text
frontend container
backend container
db container
nginx container
````

---

## 4. Docker 핵심 개념

### 4.1 Image

Image는 **실행 원본**입니다.
> 컨테이너를 만들기 위한 템플릿

예:

* `postgres:17-alpine`
* `nginx:alpine`
* `openjdk:17`

즉 이미지는 실행 가능한 원본 패키지이고,
실제로 돌아가는 것은 아닙니다.

---

### 4.2 Container

Container는 **Image를 실제로 실행한 상태**입니다.

즉:

* Image = 원본
* Container = 실행 중인 결과물

개발자 관점에서는 컨테이너를 다음처럼 이해하면 됩니다.

> 독립된 실행 공간 안에서 돌아가는 프로세스

---

### 4.3 Dockerfile

Dockerfile은 **이미지를 만드는 설명서**입니다.

예:

```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

이 파일은 다음을 정의합니다.

* 어떤 베이스 환경을 쓸지
* 어떤 파일을 복사할지
* 어떤 포트를 쓸지
* 어떤 명령으로 시작할지

즉:

> Dockerfile = 이미지를 만드는 레시피

---

### 4.4 Docker Compose

Compose는 **여러 컨테이너를 한 번에 관리하는 방식**입니다.

예를 들어 서비스가 아래처럼 구성될 수 있습니다.

* backend
* db
* frontend
* nginx

이걸 하나씩 실행하는 대신, `compose.yaml` 또는 `docker-compose.yml`에 적고 한 번에 관리합니다.

즉:

> Compose = 서비스 구조 설명서 + 멀티 컨테이너 실행 도구

---

### 4.5 Volume

Volume은 **컨테이너 밖에 따로 두는 저장공간**입니다.

이 개념이 매우 중요합니다.

왜냐하면 컨테이너는 기본적으로 “실행 공간”이기 때문에,
삭제하고 다시 만들 수 있습니다.

그런데 DB 데이터, 업로드 파일처럼 **절대 날아가면 안 되는 데이터**는 따로 보관해야 합니다.

즉:

> Volume = 컨테이너를 지워도 남겨야 하는 데이터 저장소

---

### 4.6 Docker Network

컨테이너끼리도 통신해야 합니다.

예를 들어 backend 컨테이너가 db 컨테이너에 붙어야 할 수 있습니다.

이때 중요한 포인트는:

* 컨테이너 안에서 `localhost`는 그 컨테이너 자기 자신
* 다른 컨테이너에 붙으려면 보통 Compose 서비스명으로 접근

예:

* `db`
* `backend`
* `redis`

즉 Docker Network는:

> 컨테이너끼리 통신할 수 있게 해주는 내부 네트워크

---

## 5. Docker를 비유로 이해하기

Docker를 비유하면 **밀키트**에 가깝습니다.

### Docker가 없을 때

요리 레시피만 들고 다른 집에 가서 요리하는 느낌

문제:

* 가스레인지 다름
* 냄비 다름
* 재료 없음
* 환경 다름

### Docker가 있을 때

재료, 양념, 설명서가 다 들어 있는 밀키트를 들고 가는 느낌

즉:

* 프로그램 = 요리
* Docker = 포장된 실행 세트

---

## 6. Docker 실제 사용 흐름

실제 사용 흐름은 보통 아래와 같습니다.

```text
Dockerfile → Image → Container
```

실무에서 보면 보통 이렇게 진행됩니다.

1. Dockerfile 작성
2. 이미지 빌드
3. 컨테이너 실행
4. 상태 확인
5. 로그 확인
6. 필요하면 Compose로 여러 서비스 같이 운영

---

## 7. 명령어 세트와 해석

### 7.1 이미지 빌드

```bash
docker build -t my-app .
```

#### 뜻

현재 폴더의 Dockerfile을 기준으로 `my-app` 이미지를 만든다.

#### 언제 쓰는가

* 처음 이미지 만들 때
* 코드 수정 후 새 이미지 만들 때

#### 개발자 해석

> 실행 원본을 새로 포장하는 작업

---

### 7.2 컨테이너 실행

```bash
docker run -d -p 8080:8080 --name my-app my-app
```

#### 뜻

`my-app` 이미지를 기반으로 `my-app` 컨테이너를 백그라운드에서 실행한다.

#### 세부 해석

* `-d` = 백그라운드 실행
* `-p 8080:8080` = 외부 8080과 컨테이너 8080 연결
* `--name my-app` = 컨테이너 이름 지정

#### 개발자 해석

> 이미지를 실제 실행 가능한 서비스로 띄우는 작업

---

### 7.3 실행 중인 컨테이너 확인

```bash
docker ps
```

#### 뜻

현재 살아 있는 컨테이너 목록 보기

#### 주로 보는 것

* 컨테이너 이름
* 이미지 이름
* 포트
* 상태

#### 개발자 해석

> 지금 뭐가 실제로 돌아가고 있는지 확인

---

### 7.4 전체 컨테이너 확인

```bash
docker ps -a
```

#### 뜻

종료된 컨테이너까지 포함해 전체 보기

#### 개발자 해석

> 실행했다가 죽은 컨테이너 흔적까지 확인

---

### 7.5 이미지 목록 확인

```bash
docker images
```

#### 뜻

현재 시스템에 있는 이미지 목록 보기

#### 개발자 해석

> 내가 어떤 실행 원본들을 가지고 있는지 확인

---

### 7.6 컨테이너 로그 보기

```bash
docker logs my-app
```

#### 뜻

해당 컨테이너의 로그를 출력

#### 실시간 로그 보기

```bash
docker logs -f my-app
```

#### 최근 100줄만 보기

```bash
docker logs --tail 100 my-app
```

#### 개발자 해석

> 앱이 왜 안 뜨는지, 어떤 에러가 나는지 확인하는 핵심 도구

---

### 7.7 컨테이너 내부 접속

```bash
docker exec -it my-app /bin/sh
```

또는

```bash
docker exec -it my-app /bin/bash
```

#### 뜻

실행 중인 컨테이너 안으로 직접 들어감

#### 언제 쓰는가

* 파일 확인
* 환경변수 확인
* 내부 경로 확인
* 디버깅

#### 개발자 해석

> 컨테이너 내부를 직접 들여다보는 작업

---

### 7.8 컨테이너 중지 및 삭제

```bash
docker stop my-app
```

#### 뜻

컨테이너 중지

```bash
docker rm my-app
```

#### 뜻

중지된 컨테이너 삭제

#### 개발자 해석

> 오래된 컨테이너를 정리하고 새 버전으로 다시 띄우기 위한 기본 작업

---

### 7.9 Compose 전체 실행

```bash
docker compose up -d
```

#### 뜻

Compose 파일에 정의된 전체 서비스 실행

#### 빌드 포함 실행

```bash
docker compose up -d --build
```

#### 뜻

필요한 이미지를 다시 빌드한 뒤 전체 실행

#### 개발자 해석

> backend, db, frontend 같은 여러 서비스를 한 번에 올리는 작업

---

### 7.10 Compose 상태 확인

```bash
docker compose ps
```

#### 뜻

Compose로 관리 중인 서비스 상태 확인

#### 개발자 해석

> 멀티 컨테이너 구조 전체가 지금 어떤 상태인지 보기

---

### 7.11 Compose 로그 확인

```bash
docker compose logs
```

#### 뜻

전체 서비스 로그 보기

특정 서비스만 보기:

```bash
docker compose logs backend
```

실시간 보기:

```bash
docker compose logs -f backend
```

#### 개발자 해석

> 특정 서비스의 문제를 집중 추적하는 데 사용

---

### 7.12 Compose 전체 종료

```bash
docker compose down
```

#### 뜻

Compose로 실행한 서비스 전체 종료 및 정리

#### 개발자 해석

> 구조 전체를 내리고 다시 깔끔하게 시작할 때 사용

---

## 8. Volume이란 무엇인가

Volume은 다음처럼 이해하면 됩니다.

> 컨테이너 밖에 따로 보관하는 영속 저장공간

컨테이너는 실행용 공간이라서 다시 만들 수 있습니다.
하지만 데이터는 날아가면 안 됩니다.

그래서 아래 같은 것은 Volume에 저장합니다.

* DB 데이터
* 업로드 파일
* 유지해야 하는 로그
* 설정 공유 파일

---

## 9. Volume을 왜 사용하는가

Volume을 쓰는 이유는 명확합니다.

### 1) 컨테이너를 지워도 데이터를 남기기 위해

컨테이너는 다시 만들 수 있지만, 데이터는 남아야 하기 때문입니다.

### 2) DB를 안전하게 유지하기 위해

Postgres 같은 DB는 데이터가 유지되어야 합니다.

### 3) 업로드 파일을 보존하기 위해

사용자가 올린 파일은 앱 재배포와 상관없이 남아야 합니다.

### 4) 실행 공간과 데이터 공간을 분리하기 위해

앱은 교체 가능하게, 데이터는 보존 가능하게 만드는 구조입니다.

---

## 10. Volume을 안 쓰면 생길 수 있는 문제

예를 들어 Postgres 컨테이너를 볼륨 없이 띄웠다고 가정합니다.

그러면:

* 컨테이너 안에만 DB 데이터가 저장될 수 있음
* 컨테이너를 삭제하고 다시 만들면 데이터 유실 가능

즉:

> 컨테이너만 믿고 데이터를 넣어두면 위험할 수 있음

---

## 11. Volume 사용 방식 2가지

### 11.1 Docker Named Volume

예:

```bash
-v pgdata:/var/lib/postgresql/data
```

#### 뜻

* `pgdata` = Docker가 관리하는 볼륨 이름
* `/var/lib/postgresql/data` = 컨테이너 내부 DB 저장 위치

#### 특징

* Docker가 관리
* 간편함
* DB 저장용으로 많이 사용

---

### 11.2 Bind Mount

예:

```bash
-v /data/postgres:/var/lib/postgresql/data
```

#### 뜻

* 서버의 `/data/postgres`
* 컨테이너의 `/var/lib/postgresql/data`

를 직접 연결

#### 특징

* 서버 폴더를 직접 사용
* 실제 파일 경로를 눈으로 확인하기 쉬움
* 업로드 파일, 설정 파일 연결에 자주 사용

---

## 12. Docker에서 Volume 사용하는 예시

### 예시 1. Postgres에 Docker Named Volume 사용

```bash
docker run -d \
  --name my-db \
  -e POSTGRES_PASSWORD=1234 \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15
```

#### 해석

* Postgres 컨테이너 실행
* DB 데이터는 `pgdata` 볼륨에 저장
* 컨테이너 내부에서는 `/var/lib/postgresql/data` 사용

---

### 예시 2. 서버 폴더 직접 연결

```bash
docker run -d \
  --name my-app \
  -v /home/ubuntu/uploads:/app/uploads \
  -p 8080:8080 \
  my-app
```

#### 해석

* 서버의 `/home/ubuntu/uploads`
* 컨테이너의 `/app/uploads`

를 연결

즉 컨테이너 안의 `/app/uploads`는 실제로 서버 폴더와 연결됨

---

## 13. Compose에서 Volume 사용하는 예시

### Named Volume 예시

```yaml
services:
  db:
    image: postgres:15
    container_name: my-db
    environment:
      POSTGRES_PASSWORD: 1234
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

#### 해석

* `db` 서비스는 Postgres
* DB 데이터는 `/var/lib/postgresql/data`
* 실제 저장은 `pgdata` 볼륨

---

### Bind Mount 예시

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - /data/postgres:/var/lib/postgresql/data
```

#### 해석

* 서버 폴더 `/data/postgres`
* 컨테이너의 Postgres 데이터 경로 연결

---

## 14. OCI에서 Volume을 어떻게 이해해야 하는가

OCI를 사용할 때는 `Docker Volume`과 `OCI Volume`을 구분해야 합니다.

### Docker Volume

Docker가 서버 내부에서 관리하는 저장소

### OCI Block Volume

OCI가 제공하는 별도 디스크

### OCI File Storage

OCI가 제공하는 네트워크 파일 저장소

실무적으로는 다음처럼 이해하면 됩니다.

### 방식 A. Docker Volume만 사용

OCI Compute 서버 안에 Docker가 있고, Docker가 자체적으로 볼륨을 관리

### 방식 B. OCI Block Volume을 서버에 붙이고 bind mount

OCI 디스크를 서버에 연결한 뒤, 그 경로를 컨테이너에 연결

---

## 15. Docker Volume과 OCI Volume의 차이

| 구분               | 의미                  | 관리 주체          |
| ---------------- | ------------------- | -------------- |
| Docker Volume    | Docker가 관리하는 영속 저장소 | Docker         |
| OCI Block Volume | OCI가 제공하는 별도 디스크    | Oracle Cloud   |
| Bind Mount       | 서버 폴더를 컨테이너에 연결     | 서버 OS + Docker |

즉:

* **Docker Volume**은 Docker 관점의 저장소
* **OCI Volume**은 클라우드 인프라 관점의 디스크

입니다.

---

## 16. 현재 운영 서버 구조 확인 결과

현재 운영 서버에서 확인한 결과를 기준으로 정리하면 아래와 같습니다.

### 16.1 현재 컨테이너 목록

`docker ps` 결과 기준:

* `000-frontend`
* `000-backend`
* `000-db`
* `000-nginx`

즉 구조는 대략 아래와 같습니다.

```text
nginx
  ↓
frontend / backend
  ↓
postgres(db)
```

---

### 16.2 Postgres 컨테이너 Volume 확인 결과

실행한 명령:

```bash
docker inspect 000-db --format '{{json .Mounts}}'
```

확인된 핵심 값:

```json
{
  "Type": "volume",
  "Name": "000_full_postgres_data",
  "Source": "/var/lib/docker/volumes/000-full_postgres_data/_data",
  "Destination": "/var/lib/postgresql/data"
}
```

---

### 16.3 현재 DB 저장 구조 해석

이 결과를 해석하면 다음과 같습니다.

#### `Type: volume`

* Docker Named Volume 사용 중

#### `Name: 000_full_postgres_data`

* 현재 Postgres 데이터용 볼륨 이름

#### `Destination: /var/lib/postgresql/data`

* 컨테이너 내부 Postgres 데이터 경로

#### `Source: /var/lib/docker/volumes/000-full_postgres_data/_data`

* 실제 OCI 서버(호스트)에서 데이터가 저장되는 물리 경로

즉 현재 구조는:

> Postgres 컨테이너를 따로 운영하고 있고,
> DB 데이터는 Docker Named Volume에 저장되며,
> 실제 서버 저장 위치는 `/var/lib/docker/volumes/000-full_postgres_data/_data`이다.

---

## 17. unhealthy 상태란 무엇인가

`docker ps`에서 `unhealthy`는 다음 뜻입니다.

> 컨테이너는 살아 있지만, health check 기준으로 정상 상태가 아니라고 판단된 상태

즉:

* `running` = 정상 실행 중
* `exited` = 종료됨
* `unhealthy` = 실행 중이지만 정상 점검 실패

현재 서버에서는:

* `000-backend` → unhealthy
* `000-frontend` → unhealthy

상태로 보였음

이는 보통:

* 앱 시작 실패
* health endpoint 응답 실패
* 내부 예외 발생

등과 연결될 수 있습니다.

---

## 18. 로그가 너무 많을 때 확인하는 방법

실무에서 로그는 전체를 다 읽지 않습니다.
필요한 부분만 잘라서 봅니다.

### 최근 100줄 보기

```bash
docker logs --tail 100 000-backend
```

### 실시간으로 최근 로그 보기

```bash
docker logs --tail 50 -f 000-backend
```

### 에러 키워드 검색

```bash
docker logs 000-backend 2>&1 | grep -i error
docker logs 000-backend 2>&1 | grep -i exception
docker logs 000-backend 2>&1 | grep -i failed
docker logs 000-backend 2>&1 | grep -i "caused by"
```

### 에러 주변 함께 보기

```bash
docker logs 000-backend 2>&1 | grep -i -C 5 "caused by"
```

### health check 결과 보기

```bash
docker inspect 000-backend --format '{{json .State.Health}}'
```

즉 로그를 볼 때 핵심은:

1. 최근 로그 보기
2. 에러 키워드 검색
3. healthcheck 결과 확인

---

## 19. 현재 서버에서 주로 확인할 명령어

### 컨테이너 상태 확인

```bash
docker ps
```

### Postgres 마운트 확인

```bash
docker inspect 000-db --format '{{json .Mounts}}'
```

### 볼륨 목록 확인

```bash
docker volume ls
```

### 특정 볼륨 상세 확인

```bash
docker volume inspect 000_full_postgres_data
```

### backend 최근 로그 보기

```bash
docker logs --tail 100 000-backend
```

### backend 에러 원인 검색

```bash
docker logs 000-backend 2>&1 | grep -i -C 5 "caused by"
```

### backend health 확인

```bash
docker inspect 000-backend --format '{{json .State.Health}}'
```

### 컨테이너 내부 접속

```bash
docker exec -it 000-backend /bin/sh
docker exec -it 000-db /bin/sh
```

---

## 20. 개발자가 꼭 알아야 할 핵심 정리

### Docker 관련

* **Image** = 실행 원본
* **Container** = 실행 중인 상태
* **Dockerfile** = 이미지 만드는 설명서
* **Compose** = 여러 컨테이너를 함께 관리하는 구조
* **Volume** = 데이터를 남기기 위한 저장소
* **Network** = 컨테이너 간 통신 경로

### 운영 구조 관련

* 컨테이너는 다시 만들 수 있음
* 데이터는 Volume으로 분리해야 안전함
* DB 데이터는 보통 컨테이너 안이 아니라 Volume에 둠
* `localhost`는 현재 위치 자기 자신임
* 컨테이너 안에서 `localhost`는 그 컨테이너 자신임

### 현재 서버 기준

* Postgres는 별도 컨테이너로 운영 중
* DB 데이터는 Docker Named Volume 사용 중
* 볼륨 이름은 `000_full_postgres_data`
* 실제 저장 위치는 `/var/lib/docker/volumes/000-full_postgres_data/_data`

---

## 21. 결론

중요한 포인트는 아래와 같습니다.

* 컨테이너는 실행 공간이다.
* 데이터는 별도로 보존해야 한다.
* 그 역할을 하는 것이 Volume이다.
* 현재 운영 서버에서는 Postgres가 별도 컨테이너로 돌고 있으며, Docker Named Volume을 사용 중이다.
* 장애 확인 시에는 상태 확인 → 로그 확인 → health check 확인 → volume/mount 확인 순으로 접근하면 된다.

즉 지금 단계에서 최소한 아래 정도는 이해하고 있어야 합니다.

> “현재 시스템은 Docker 컨테이너들로 구성되어 있고,
> Postgres 데이터는 컨테이너 안이 아니라 Docker Volume에 보관되고 있으며,
> 문제가 생기면 상태 / 로그 / 마운트 / health check를 확인해야 한다.”

* 마운트(mount) = 어떤 저장공간이나 폴더를 다른 곳에 연결해서 보이게 하는 것
>  Docker에서는 보통 서버의 폴더/볼륨을 컨테이너 안 경로에 연결하는 것

---
예를 들어

서버 폴더: /data/postgres

컨테이너 경로: /var/lib/postgresql/data

> 이렇게 연결하면 컨테이너는 자기 안에 저장하는 것처럼 보이지만, 실제 데이터는 서버 쪽 저장공간에 남아
> 컨테이너를 지워도 데이터가 안 날아가게 만들 때 마운트를 많이 사용
>마운트 = 저장공간을 연결해서 다른 위치에서도 같은 데이터가 보이게 하는 것