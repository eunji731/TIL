## 📌 RestController의 return 값 형식 정리

---

## 1) RestController에서 return이 의미하는 것

RestController에서 return은 **“프론트에게 보내는 최종 응답”**이다.

- 🖥️ JSP처럼 화면 이름을 반환하는 것이 아니라
- 📦 **데이터 자체를 반환**한다.

즉,

👉 **return 값 = 프론트가 실제로 받는 값**이다.

---

## 2) RestController에서 return 할 수 있는 것들

RestController에서는 크게 **세 가지 형태**로 return 한다.

- 📦 데이터만 return
- ✉️ ResponseEntity를 return
- ✉️ + 📦 ResponseEntity + ApiResponse를 return

이 **세 가지만** 이해하면 된다.

---

## 3) 데이터만 return 하는 경우

### 의미

- 서버가 데이터를 그대로 보낸다
- 성공으로 간주된다

### 특징

- 상태 표시는 항상 성공으로 처리된다
- 실패나 권한 없음 표현이 어렵다

### 사용 상황

- 🧪 테스트용
- 🔍 아주 단순한 조회

---

## 4) ResponseEntity를 return 하는 경우

### 의미

- 데이터와 함께
- 이 요청이 성공인지 실패인지 **명확히 표시한다**

### 특징

- 성공 / 실패 / 권한 없음 등을 구분할 수 있다
- 프론트에서 상태에 따라 분기 처리 가능하다

### 사용 상황

- 🌐 실제 서비스 API 대부분
- ⚠️ 상태 코드가 중요한 경우

---

## 5) ResponseEntity + ApiResponse를 return 하는 경우

### 의미

- 상태 표시와
- 응답 데이터의 모양까지
- 모두 통제한다

### 구조적으로 보면

- ✉️ **ResponseEntity**
    - 성공인지 실패인지 **상태 담당**
- 📦 **ApiResponse**
    - 응답 내용의 **형태 담당**

즉,

👉 **“상태는 ResponseEntity로, 내용물 모양은 ApiResponse로”** 관리하는 방식이다.

---

## 6) 가장 많이 쓰는 return 형식

실무에서 가장 흔한 패턴은 다음과 같다.

- ✅ 성공일 때
    - 성공 상태 표시
    - ApiResponse 형태로 데이터 전달
- ❌ 실패일 때
    - 실패 상태 표시
    - ApiResponse 형태로 메시지 전달

이렇게 하면

✨ **모든 API 응답이 같은 형태를 가지게 된다.**

---

## 7) return 값이 JSON이 되는 이유

RestController에서는 return 한 객체를

🔄 **자동으로 JSON으로 변환**해서 보낸다.

- 개발자는 JSON을 직접 만들지 않는다.
- 객체를 return 하면 Spring이 알아서 JSON으로 바꿔 프론트에 전달한다.

---

## 8) 왜 Entity를 그대로 return 하면 안 되냐?

Entity를 그대로 return 하면 다음 문제가 생길 수 있다 ⚠️

- 필요 없는 데이터가 같이 나갈 수 있고
- 내부 구조가 그대로 노출될 수 있고
- 나중에 구조 변경이 매우 힘들어진다

그래서 보통은

- Entity → 🎯 Response용 객체로 변환
- 그 객체를 return 한다

이 방식이 **안전하고 유지보수에 유리**하다.

---

## 9) 한 줄 요약

RestController의 return 값은 **프론트에게 전달되는 최종 데이터**이며,

실무에서는 보통

- ✉️ ResponseEntity로 상태를 표시하고
- 📦 ApiResponse로 응답 형태를 통일해서

return 한다.