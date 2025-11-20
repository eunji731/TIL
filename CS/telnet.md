# Telnet

## 1. Telnet?
**Telnet(Telecommunication Network)** 은 원격 시스템에 텍스트 기반으로 접속하기 위한 네트워크 프로토콜.  
TCP 기반으로 동작하며, 예전에는 서버 관리에 많이 사용되었으나 **암호화가 없어 보안에 매우 취약**하여 현재는 거의 사용되지 않음.  
(대부분 **SSH**가 대체)

---

## 2. Telnet의 특징

### ✔ 암호화 없음 (보안 취약)
- 아이디, 비밀번호, 명령어까지 모두 평문으로 전송됨  
- 패킷 스니핑에 그대로 노출됨

### ✔ 기본 포트: 23번
- Telnet 서버가 사용하는 기본 포트는 **TCP 23**

### ✔ 텍스트 기반 원격 제어
- 연결되면 원격 장비의 명령어를 실행할 수 있음  
- 서버, 라우터, 스위치 등 네트워크 장비 관리에 사용되기도 함

---

## 3. Telnet의 주요 용도 (현재 실무 기준)
Telnet은 관리용으로는 거의 쓰지 않지만, **네트워크 테스트용**으로 매우 많이 사용된다.

### ✔ 1) 포트 열림 여부 확인 (가장 많이 사용)

예시 )
telnet 192.168.0.10 8080
- 연결됨 → 해당 포트 열려 있음  
- 연결 실패 → 방화벽 차단 / 서버 다운 / 포트 미개방

### ✔ 2) 서버 TCP 응답 테스트
SMTP, Redis, MySQL, Tomcat 등의 서비스가 살아 있는지 빠르게 확인할 때 사용.

예: SMTP 테스트
telnet smtp.gmail.com 587


### ✔ 3) 방화벽 / 포트포워딩 점검
- 서버 간 통신 문제 발생 시  
- 방화벽 설정이 맞는지 확인할 때 유용함

---

## 4. Telnet 설치 방법

### ■ Windows
dism /online /Enable-Feature /FeatureName:TelnetClient

또는  
제어판 → 프로그램 → Windows 기능 켜기/끄기 → **Telnet Client 체크**

### ■ Linux
Ubuntu/Debian:

sudo apt install telnet

CentOS/RHEL:

sudo yum install telnet


---

## 5. Telnet 사용 방법

### ✔ 접속


telnet <IP> <PORT>

예:


telnet 10.0.0.5 8080


### ✔ 접속 성공 메시지


Connected to 10.0.0.5.
Escape character is '^]'.


### ✔ 종료
- `Ctrl + ]`
- `quit`

---

## 6. Telnet을 사용하면 안 되는 상황
- 서버에 로그인할 때 (보안 이슈)
- 외부 네트워크 환경
- SSH가 가능한 모든 환경

→ 반드시 **SSH** 사용:


ssh user@server


---

## 7. Telnet vs SSH 비교

| 항목 | Telnet | SSH |
|------|--------|------|
| 보안 | ❌ 평문 전송 | ✔️ 암호화 |
| 기본 포트 | 23 | 22 |
| 사용 목적 | 테스트 & 진단 | 서버 원격 접속 |
| 실무 사용 빈도 | 거의 없음 | 필수 |

---

## 8. 정리
- Telnet은 오래된 원격 접속 프로토콜  
- 보안 취약 때문에 SSH로 대체  
- **현재는 주로 포트 테스트용으로 사용됨**  