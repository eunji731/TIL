# SSL 정리 문서

## 1. SSL 이란?
SSL(Secure Sockets Layer)은 인터넷에서 데이터를 안전하게 전송하기 위해 사용하는 보안 프로토콜
현재 실제 표준은 SSL이 발전한 TLS(Transport Layer Security)이며, 관습적으로 SSL이라는 명칭을 함께 사용

---

## 2. SSL 이 제공하는 보안 기능

### 2.1 기밀성(Confidentiality)
데이터를 암호화하여 중간에서 내용을 볼 수 없도록 한다.

### 2.2 무결성(Integrity)
전송 중 데이터가 변조되었는지 여부를 확인한다.

### 2.3 인증(Authentication)
접속하려는 서버가 신뢰 가능한 서버인지 확인한다.

---

## 3. SSL/TLS 버전 변천

- SSL 1.0: 공개되지 않음  
- SSL 2.0: 보안 취약점으로 폐기  
- SSL 3.0: 사용 중단  
- TLS 1.0, 1.1: 구버전, 비권장  
- TLS 1.2: 현재 가장 널리 사용  
- TLS 1.3: 최신 버전, 단순하고 빠르며 안전함

---

## 4. SSL에서 사용되는 암호 기술

### 4.1 대칭키 암호화
- 예: AES, ChaCha20  
- 실제 데이터(HTTP 본문 등)는 모두 대칭키로 암호화  
- 암호화/복호화가 빠르다

### 4.2 비대칭키 암호화
- 예: RSA, ECDHE, ECDSA  
- 공개키/개인키 구조  
- 인증서 검증 및 세션키 교환에서 사용

### 4.3 해시와 HMAC
- 예: SHA256, HMAC-SHA256  
- 데이터 변조 여부를 검사한다.

---

## 5. SSL/TLS 프로토콜 구조

### 5.1 Handshake Protocol
- 서버 인증  
- 키 교환  
- 암호 알고리즘 협상

### 5.2 Record Protocol
- 실제 애플리케이션 데이터 전송  
- 대칭키로 암호화  
- 무결성 검증 포함

---

## 6. SSL Handshake 흐름 (TLS 1.2 기준)

1. 클라이언트가 ClientHello 전송  
   - 지원 프로토콜 버전  
   - 지원 암호 스위트 목록  
   - client_random 값 전송

2. 서버가 ServerHello 전송  
   - 선택된 프로토콜 버전  
   - 선택된 암호 스위트  
   - server_random 값 전송

3. 서버가 인증서(Certificate) 전송  
   - 서버 공개키 포함  
   - 인증서 체인 포함

4. 클라이언트가 인증서 검증  
   - 신뢰할 수 있는 CA 여부  
   - 도메인 일치 여부  
   - 유효기간 확인  
   - 폐기 여부 확인

5. 키 교환(ECDHE 등) 수행  
   - 클라이언트와 서버는 공유 비밀키(shared secret) 생성

6. 세션키 생성  
   - 랜덤값들과 shared secret을 조합해 대칭키 생성

7. ChangeCipherSpec 교환  
   - 이제부터 암호화 사용 선언  
   - 이후 데이터는 Record Protocol을 통해 암호화되어 전송됨

---

## 7. SSL 인증서의 역할

### 7.1 인증서가 포함하는 정보
- 도메인 정보(CN, SAN)  
- 서버 공개키  
- 발급자(CA)  
- 유효기간  
- 서명 알고리즘  
- 인증서 용도 정보

### 7.2 인증서 체인 구조
- Leaf Certificate (사이트 인증서)  
- Intermediate CA 인증서  
- Root CA 인증서 (OS/브라우저에 내장)

클라이언트는 이 체인을 검증하여 서버를 신뢰할 수 있는지 판단한다.

---

## 9. 세션 재개(Session Resumption)

### 9.1 Session ID 방식
서버가 Session ID를 발급하고, 클라이언트가 다음 접속 시 이를 제시하여 세션을 재활성화한다.

### 9.2 Session Ticket 방식
세션 정보를 암호화해서 클라이언트가 보관하며, 서버는 티켓을 복호화하여 세션을 복원한다.

---

## 10. HTTPS와 SSL의 관계

- SSL/TLS는 전송 계층 보안 프로토콜  
- HTTPS는 HTTP를 SSL/TLS 위에서 사용하는 방식  
- HTTPS는 기본적으로 443 포트를 사용하며 HTTP 데이터를 암호화하여 전송한다.

---

## 11. SSL 설정에서의 보안 고려 사항

### 11.1 사용 금지해야 할 버전
- SSL 2.0, SSL 3.0  
- TLS 1.0, TLS 1.1

### 11.2 취약한 암호 스위트 제외
- RC4  
- 3DES  
- EXPORT 등 약한 암호  
- NULL 암호

### 11.3 Forward Secrecy
ECDHE 등 ephemeral 방식의 키 교환을 사용하여 과거 트래픽 보호.

---

## 12. SSH와 SSL의 차이

| 항목 | SSL/TLS | SSH |
|------|---------|-----|
| 목적 | 데이터 전송 보호 | 원격 접속/명령 실행 |
| 주요 사용처 | HTTPS | SSH, SFTP, SCP |
| 인증 방식 | CA 기반 인증서 | 사용자 공개키 |
| 포트 | 443 | 22 |
| 구조 | X.509 인증서 체계 | CA 불필요, SSH keypair 기반 |

---

## 13. 요약

- SSL은 데이터를 안전하게 전송하기 위한 보안 프로토콜
- 실제 사용되는 표준은 TLS이며 일반적으로 SSL이라는 이름으로 함께 사용
- SSL은 기밀성, 무결성, 인증을 제공
- HTTPS는 HTTP를 SSL/TLS 위에서 사용하는 프로토콜
- SSH는 SSL과 목적과 구조가 다른 원격 접속 프로토콜
- 실무에서는 TLS 1.2 또는 TLS 1.3을 사용

