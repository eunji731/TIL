# React(Vite) - Spring Boot - Supabase 아키텍처 연동 및 CORS 해결 

## 💡 학습 목표
- Vite 기반의 React 프론트엔드와 Spring Boot 백엔드 간의 통신 구조 이해
- Supabase를 활용한 DB/인증 계층 분리 개념 확립
- 안티그래비티(Antigravity)를 활용한 바이브 코딩(Vibe Coding)으로 프론트-백-DB 연동 파이프라인 구축
- CORS 에러의 원인 파악 및 백엔드에서의 해결 방법

## 🏗️ 전체 시스템 아키텍처
현재 구성한 시스템은 세 가지 주요 계층으로 나뉜다.


1. **Vite (React) - UI 및 클라이언트 상태 관리**: 사용자 인터페이스를 담당하며, 포트 `5173`에서 실행된다.
2. **Spring Boot - 비즈니스 로직 및 API**: 핵심 비즈니스 로직과 데이터 검증을 담당하며, 포트 `8080`에서 실행된다.
3. **Supabase - BaaS (DB & Auth)**: PostgreSQL 기반의 데이터베이스와 인증(Authentication)을 제공한다.

단순 데이터 조회나 인증은 React에서 Supabase로 직접 통신(Direct Path)하고, 복잡한 로직이 필요한 데이터 처리는 Spring Boot를 거쳐 통신(Logic Path)하는 유연한 구조를 채택했다.

## 🚀 안티그래비티 기반 프론트엔드-백엔드 연동의 핵심

### 1. CORS (Cross-Origin Resource Sharing) 문제와 해결
프론트엔드(`localhost:5173`)에서 백엔드(`localhost:8080`)로 API 요청을 보낼 때, 출처(Origin)가 달라 브라우저 정책상 CORS 에러가 발생한다.
이를 프론트의 프록시 설정으로 우회할 수도 있지만, 정석적인 방법은 **백엔드(Spring Boot)에서 프론트엔드의 접근을 허용**해 주는 것이다.

- **해결책**: Spring Boot의 `WebMvcConfigurer`를 구현하여 전역 CORS 설정을 추가.
- `allowedOrigins("http://localhost:5173")`를 명시하여 Vite 서버의 요청을 안전하게 수락하도록 설정함.

### 2. 통신 모듈 분리 (관심사의 분리)
안티그래비티 등 AI 어시스턴트와 함께 코드를 작성할 때, 통신 로직이 컴포넌트 내부에 산재해 있으면 유지보수와 컨텍스트 파악이 매우 어렵다. 따라서 두 가지 통신 인스턴스를 명확히 분리했다.

- `apiClient.ts`: Axios 인스턴스. Spring Boot API 전용 (기본 URL 및 헤더 공통 설정)
- `supabaseClient.ts`: Supabase Client. 직접 DB/Auth 통신용

## 📝 느낀 점
프론트엔드부터 백엔드, DB까지 전체 흐름을 연결하는 뼈대를 세웠다. 각 기술 스택의 역할을 명확히 구분하고 통신 모듈을 모듈화해두니, 이후 기능 개발 시 안티그래비티에 컨텍스트를 제공하거나 프롬프팅하기 훨씬 수월한 구조가 되었다.