# BufferedReader, BufferedReader

## 1. BufferedReader란?

### ✔ 한 줄 전체를 빠르게 읽는 입력 도구
- `Scanner`보다 훨씬 빠름 → 코테에서 필수
- 입력을 **문자열(String)** 형태로 통째로 읽어옴
- 읽은 문자열을 숫자로 바꿔서 사용함

### 📌 기본 선언
```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in))
String line = br.readLine();
```
---
## 2. StringTokenizer란?
- 읽어온 문자열을 “공백 기준으로” 잘라주는 도구
- "10 20 30" 형태의 문자열을 -> "10", "20", "30" 이렇게 잘라줌
- 반복하면서 하나씩 꺼낼 수 있음

```java
StringTokenizer st = new StringTokenizer("10 20 30");
st.nextToken();  // "10"
st.nextToken();  // "20"
st.nextToken();  // "30"
```
---
## 3. BufferedReader + StringTokenizer 조합
```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
StringTokenizer st;

// 정수 하나 입력
int n = Integer.parseInt(br.readLine());

// 한 줄에 여러 정수 입력
st = new StringTokenizer(br.readLine());
int a = Integer.parseInt(st.nextToken());
int b = Integer.parseInt(st.nextToken());
```
---
## 4. 배열 입력 템플릿
```java
int n = Integer.parseInt(br.readLine());
int[] arr = new int[n];

st = new StringTokenizer(br.readLine());
for (int i = 0; i < n; i++) {
    arr[i] = Integer.parseInt(st.nextToken());
}
```
---
## 5. 둘의 역할 요약

| 도구               | 역할               |
|--------------------|--------------------|
| **BufferedReader** | 한 줄 전체 읽기    | 
| **StringTokenizer**| 공백 기준 잘라내기 | 

---
## 6. 전체 흐름도
입력: "10 20 30"
        ↓
BufferedReader.readLine()  →  "10 20 30"
        ↓
StringTokenizer            →  "10" → "20" → "30"
        ↓
Integer.parseInt()         →  10, 20, 30

---
## 정리
- BufferedReader = 한 줄 읽기
- StringTokenizer = 공백 기준으로 자르기
- nextToken() = 하나씩 꺼내기