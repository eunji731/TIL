# SSH (Secure Shell)

## 1. SSH?
- SSH(Secure Shell)는 **네트워크를 통해 다른 컴퓨터에 안전하게 접속하기 위한 보안 프로토콜**이다.
- 암호화되지 않은 Telnet, rlogin 등의 원격 접속 방식의 보안 취약점을 보완하기 위해 등장했다.
- 기본적으로 **포트 22번**을 사용한다.

## 2. SSH의 핵심 목적
- **원격 접속(Remote Login)**
- **명령 실행(Remote Command Execution)**
- **파일 전송(SCP, SFTP 기반)**
- **터널링(Tunneling, Port Forwarding)**

## 3. SSH의 동작 원리
### 3.1 클라이언트-서버 구조
- SSH 서버(sshd)가 원격 시스템에서 동작.
- SSH 클라이언트가 서버로 접속 요청.
- 양측이 암호화된 통신 채널을 생성.

### 3.2 암호화 방식
- **대칭키 암호화**  
  - 통신 중 데이터 암호화에 사용.
- **비대칭키 암호화(공개키/개인키)**  
  - 인증 과정에 사용.
- **해시(HMAC)**  
  - 메시지 무결성 확인.

### 3.3 인증 방식
- **비밀번호 인증**
- **공개키 인증 (RSA, ED25519 등)**
  - 개인키를 클라이언트가 보유, 공개키는 서버에 저장.
  - 가장 안전하고 권장되는 방식.

## 4. SSH를 사용하는 이유
- 네트워크 구간의 모든 데이터가 암호화됨.
- 패킷 가로채기·중간자 공격(MITM) 방지.
- 시스템 간 자동화·배포·관리에도 활용 가능.
- 여러 기능(SFTP, SCP, 포트 포워딩 등)을 포함한 **보안 통신 표준 프로토콜**.

## 5. SSH의 주요 기능
### 5.1 원격 접속
ssh user@host

shell
코드 복사

### 5.2 공개키 기반 접속
ssh-copy-id user@host

markdown
코드 복사

### 5.3 포트 포워딩
- 로컬 포트 → 원격 포트
ssh -L 8080:localhost:80 user@host

shell
코드 복사

### 5.4 프록시/점프호스트 접속
ssh -J jump@test.org user@host

markdown
코드 복사

## 6. SSH와 관련된 프로토콜
- **SCP(Secure Copy)**  
  - SSH 기반의 간단한 파일 복사 프로토콜.
- **SFTP(SSH File Transfer Protocol)**  
  - SSH 기반의 파일 관리·전송 프로토콜.