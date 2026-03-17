# Jira MCP with Codex CLI 정리

## 개요
이번에 정리한 내용은 **Codex CLI**와 **Atlassian Jira MCP(Rovo MCP Server)** 를 연결해서, CLI에서 Jira를 읽고 이슈를 생성/수정하는 방법이다.

이 방식의 핵심은 단순히 Jira API를 한 번 호출하는 것이 아니라, **AI가 현재 Jira 상태를 읽고 그 상태를 바탕으로 다음 작업을 이어서 제안하거나 생성할 수 있다**는 점이다.

---

## 개념 정리

### Codex CLI
터미널에서 사용하는 OpenAI Codex 기반 CLI 도구이다.

**주요 역할:**
* 로컬 파일 읽기
* 파일 수정
* 명령 실행
* MCP 서버 연결
* 외부 도구와 상호작용

### MCP
**MCP(Model Context Protocol)**는 AI가 외부 시스템의 **데이터**와 **도구**를 사용할 수 있게 해주는 연결 방식이다. AI가 단순히 텍스트만 읽는 것이 아니라 다음과 같은 작업을 할 수 있게 해준다.



* Jira 프로젝트 조회
* Jira 이슈 검색
* Jira 이슈 생성
* Jira 이슈 수정

### Jira MCP
Atlassian Rovo MCP Server를 통해 Jira를 Codex 같은 AI 클라이언트에 연결하는 방식이다. 이걸 연결하면 AI가 다음을 수행할 수 있다.
* 현재 보드 상태를 읽기
* 이미 있는 이슈 확인
* 중복되지 않게 새 작업을 제안
* 필요 시 실제 생성까지 수행

### REST API 방식과 MCP 방식 차이

| 구분 | REST API만 사용할 때 | MCP를 사용할 때 |
| :--- | :--- | :--- |
| **작동 방식** | 한 번 요청해서 결과 생성 | AI가 Jira를 도구처럼 직접 사용 |
| **상태 유지** | 현재 상태를 읽으려면 별도 로직 필요 | 현재 상태를 보며 이어서 작업 가능 |
| **적합성** | 자동화 스크립트에 적합 | 대화형 반복 작업에 적합 |

> **요약:** **한 번 생성**은 REST API도 충분하고, **계속 보면서 이어서 생성**하려면 MCP가 더 적합하다.

---

## 사용 목적
* 원페이저/요구사항 문서를 바탕으로 Jira 구조 자동 생성
* 현재 보드 상태를 보며 부족한 작업만 이어서 추가
* 중복 생성 방지
* Epic / Task / Sub-task 구조 자동화
* 일정/담당은 사람이 직접 하고 작업 구조는 AI가 보조

---

## 언제 유용한가
* 새 프로젝트 시작 시 초기 Jira 구조를 빠르게 만들고 싶을 때
* 문서 기반으로 업무를 쪼개고 싶을 때
* 큰 Epic 아래 필요한 Task/Sub-task를 체계적으로 만들고 싶을 때
* 이미 생성된 보드를 보면서 빠진 작업만 추가하고 싶을 때
* 혼자 프로젝트를 하더라도 구조적으로 관리하고 싶을 때

---

## 기본 연결 흐름
1. Codex CLI 설치
2. Atlassian API token 발급
3. `email:api_token` 값을 Base64로 인코딩
4. PowerShell 환경변수에 인증 헤더 저장
5. 프로젝트에 `.codex/config.toml` 생성
6. Codex 실행
7. `/mcp`로 연결 상태 확인
8. Jira 프로젝트 조회
9. 문서를 기준으로 초안 생성
10. 확인 후 실제 생성

---

## 사용 방법

### 1. API token 준비
Atlassian에서 API token을 만든다.
* **주의:** Rovo MCP 용도에 맞는 token인지, Jira 읽기/쓰기 접근 권한이 있는지 확인한다.

### 2. Base64 생성
형식: `이메일:API_TOKEN`

**PowerShell 예시:**
```powershell
$pair = "user@example.com:API_TOKEN"
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($pair))
```

### 3. 환경변수 저장
* **영구 저장:** `setx ATLASSIAN_AUTH_HEADER "Basic Base64값전체"`
* **현재 세션 즉시 반영:** `$env:ATLASSIAN_AUTH_HEADER = "Basic Base64값전체"`

### 4. Codex MCP 설정
`.codex/config.toml` 파일 생성 및 작성:
```toml
[mcp_servers.atlassian]
url = "https://mcp.atlassian.com/v1/mcp"
env_http_headers = { Authorization = "ATLASSIAN_AUTH_HEADER" }
required = true
```

### 5. Codex 실행
반드시 해당 프로젝트 폴더에서 실행한다.
```bash
codex
```
그 후 `/mcp` 명령어로 연결 확인.

### 6. 연결 확인 후 조회
> **프롬프트 예시:** "내가 접근 가능한 Jira 프로젝트 목록을 보여줘. 아직 아무것도 생성하지 마."

### 7. 생성 흐름
**조회 → 초안 제안 → Epic 생성 → Task 생성 → Sub-task 생성** 순서를 권장하며, 점진적으로 넣는 것이 안전하다.

#### 잘 쓰는 프롬프트 패턴
* **초안만 제안:** "현재 Jira 프로젝트 상태를 확인하고, 중복되지 않게 Epic / Task / Sub-task 초안을 제안해줘. 아직 생성하지 마."
* **Epic 먼저 생성:** "방금 제안한 Epic만 먼저 생성해줘. Task와 Sub-task는 생성하지 마."
* **하위 작업 생성:** "방금 생성한 Epic 아래에 Task와 Sub-task를 생성해줘. 중복은 피하고, description도 넣어줘."

---

## 시행착오 요약

### 1. config.toml 파일을 실행하려고 함
* **문제:** PowerShell에 파일 경로만 입력해서 실행 시도.
* **원인:** `.toml`은 실행 파일이 아닌 설정 파일임.
* **해결:** `notepad .codex\config.toml` 등으로 편집기에서 수정.

### 2. No MCP servers configured
* **문제:** `/mcp` 호출 시 서버가 안 보임.
* **원인:** Codex를 프로젝트 폴더가 아닌 다른 위치에서 실행.
* **해결:** 반드시 `.codex/config.toml`이 있는 폴더에서 실행.

### 3. setx 후 바로 값이 안 보임
* **문제:** 환경변수 저장 후 현재 창에서 확인 불가.
* **원인:** `setx`는 새 터미널 세션부터 반영됨.
* **해결:** 새 PowerShell 창을 열거나 `$env:ATLASSIAN_AUTH_HEADER = "Basic ..."`으로 직접 할당.

### 4. Base64 끝의 = / == 를 빼먹음
* **문제:** 인코딩 값 끝을 잘라냄.
* **원인:** 불완전한 복사.
* **해결:** 끝까지 전부 포함해서 사용.

### 5. MCP는 연결됐는데 Jira 툴이 적게 보임
* **원인 후보:** API token 방식 제한, 권한 부족, 잘못된 token 종류.
* **해결:** Jira read/write 권한 재확인 및 점검.

### 6. You don't have permission to connect via API token
* **문제:** 프로젝트 조회/생성 시 권한 오류.
* **원인:** 조직 설정에서 API token 기반 접근 제한 가능성.
* **해결:** Admin 설정 확인 및 필요 시 OAuth 방식 검토.

---

## 실전 팁
* 생성 전에 항상 **조회 → 초안 → 생성** 순서를 지킨다.
* 처음에는 Epic만 먼저 생성하는 방식이 안전하다.
* 모든 Task에 Sub-task를 과하게 달지 않는다.
* `description`은 가능하면 상세히 넣는다.
* 문서 기반 자동 생성 후 마지막 검토는 사람이 한다.

---

## 정리
1. Codex CLI + Jira MCP로 Jira를 대화형으로 다룰 수 있다.
2. 현재 보드 상태를 보며 중복 없는 작업 생성이 가능하다.
3. 연결 자체와 Jira 권한은 별개로 봐야 한다.
4. 환경변수, 프로젝트 경로, config 파일 위치가 매우 중요하다.
5. 실제 운영은 **조회 → 초안 → 생성** 흐름이 가장 안전하다.
