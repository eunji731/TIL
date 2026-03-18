# 🚀 Spring Boot + Supabase 연동 트러블슈팅 정리

## 📚 목차
- [문서 목적](#1-문서-목적)
- [문제 상황 요약](#2-문제-상황-요약)
- [발생했던 주요 오류](#3-발생했던-주요-오류)
- [원인 분석 과정](#4-원인-분석-과정)
- [실제 원인](#5-실제-원인)
- [오류별 정리](#6-오류별-정리)
- [해결 방법 요약](#7-해결-방법-요약)
- [재발 방지 포인트](#8-재발-방지-포인트)
- [정리](#9-정리)

---

## 1. 문서 목적
이 문서는 Spring Boot와 Supabase를 연동하면서 실제로 겪은 오류와 원인, 해결 방법을 정리한 문서입니다.  
개념 설명보다는 **실제 오류 원인과 해결 경험을 기록하는 데 초점**을 둡니다.

---

## 2. 문제 상황 요약
초기 설정 단계에서 지속적인 DB 연결 실패가 발생했습니다.  

겉으로는 `.env`나 `application-dev.yml` 설정이 적용되지 않는 것처럼 보였으나,  
조사 결과 **외부 설정의 간섭**이 주요 원인이었습니다.

---

## 3. 발생했던 주요 오류

```plaintext
org.hibernate.exception.JDBCConnectionException: unable to obtain isolated JDBC connection
Caused by: org.postgresql.util.PSQLException: The connection attempt failed.
Caused by: java.net.SocketTimeoutException: Connect timed out
```
### 상태
- 네트워크 차단
- 호스트 정보 오류
- 설정 파일 미적용

등 다양한 가능성이 의심되는 상황

---

## 4. 원인 분석 과정

- **네트워크 포트 확인**  
  → PowerShell `Test-NetConnection` 결과, 5432 및 6543 포트 모두 정상

- **DBeaver 연결 테스트**  
  → 외부 툴에서는 정상 연결  
  → DB 정보 자체는 문제 없음

- **YAML 구조 확인**  
  → 키 중복 / profile 덮어쓰기 여부 점검

- **.env 경로 확인**  
  → Working Directory와 파일 위치 일치 여부 확인

---

## 5. 실제 원인

👉 **IntelliJ Run Configuration의 Environment Variables**

- 이전에 입력해둔 환경 변수 값이 남아 있었음
- 해당 값이 `.env` 및 `yml`보다 우선 적용
- 결과적으로 최신 설정이 덮어쓰기됨

---

## 6. 오류별 정리

- **네트워크 문제 (Timeout)**  
  → 실제 네트워크 문제 아님  
  → 실행 환경 설정 문제

- **DB 정보 오류**  
  → DBeaver 정상 접속으로 배제

- **설정 파일 충돌**  
  → IDE 환경 변수가 최우선 적용됨 확인

---

## 7. 해결 방법 요약

- IntelliJ Run Configuration 확인  
  - Environment Variables 불필요 값 제거

- YAML 역할 분리  
  - 공통 / dev 설정 명확히 구분

- `.env` 사용  
  - 민감 정보 외부 분리

---

## 8. 재발 방지 포인트

- ✅ **외부 툴 교차 검증**  
  - DBeaver는 되는데 Spring만 안 되면 → 실행 환경 의심

- ✅ **IDE 설정 점검 습관화**  
  - IntelliJ Environment Variables 확인

- ✅ **경로 일치 확인**  
  - `.env` 위치 vs Working Directory

---

## 9. 정리

이번 이슈는 DB 자체 문제가 아니라  
👉 **IDE 실행 환경이 설정을 덮어쓴 사례**

따라서 트러블슈팅은 다음 순서로 접근하는 것이 효과적이다:

> 네트워크 → 외부 툴 → IDE 환경 설정