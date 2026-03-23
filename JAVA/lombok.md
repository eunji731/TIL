# Lombok 정리

## 1. Lombok이란

Lombok은 **자바에서 반복적으로 작성하는 보일러플레이트 코드를 자동 생성해주는 라이브러리**이다.

자바에서는 다음과 같은 코드가 자주 반복된다.

- getter / setter
- 생성자
- `toString()`
- `equals()` / `hashCode()`
- builder 패턴 코드

이런 코드는 기능적으로 필요하지만, 매번 직접 작성하면 코드가 길어지고 생산성이 떨어진다.  
Lombok은 이런 반복 코드를 줄여서 **핵심 로직 중심으로 코드를 작성할 수 있게 돕는 도구**이다.

---

## 2. Lombok의 목적

Lombok의 주요 목적은 다음과 같다.

### 2.1 반복 코드 감소
직접 작성해야 하는 getter, setter, 생성자 등의 양을 줄인다.

### 2.2 가독성 향상
핵심 비즈니스 로직만 보이게 하여 코드가 덜 복잡해 보이게 한다.

### 2.3 생산성 향상
반복 코드 작성 시간을 줄여 개발 속도를 높인다.

### 2.4 실수 감소
수작업으로 작성할 때 생길 수 있는 누락, 오타, 잘못된 필드 매핑을 줄인다.

---

## 3. Lombok의 대표 기능

## 3.1 `@Getter`
필드에 대한 getter 메서드를 자동 생성한다.

### 예시
```java
@Getter
private String name;
```

자동 생성되는 형태:
```java
public String getName() {
    return name;
}
```
## 3.2 @Setter

필드에 대한 setter 메서드를 자동 생성한다.

예시
```java
@Setter
private String name;
```

자동 생성되는 형태:
```java
public void setName(String name) {
    this.name = name;
}
```
## 3.3 @NoArgsConstructor

파라미터가 없는 기본 생성자를 자동 생성한다.

예시
```java
@NoArgsConstructor
public class User {
}
```
자동 생성되는 형태:
```java
public User() {
}
```
## 3.4 @AllArgsConstructor

모든 필드를 파라미터로 받는 생성자를 자동 생성한다.

예시
```java
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```
자동 생성되는 형태:
```java
public User(String name, int age) {
    this.name = name;
    this.age = age;
}
```
## 3.5 @RequiredArgsConstructor

final 필드와 @NonNull 필드를 파라미터로 받는 생성자를 자동 생성한다.

예시
```java
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
}
```
자동 생성되는 형태:
```java
public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
}
```
## 3.6 @ToString

객체의 필드값을 문자열로 보여주는 toString() 메서드를 자동 생성한다.

예시
```java
@ToString
public class User {
    private String name;
}
```
## 3.7 @EqualsAndHashCode

객체 비교를 위한 equals()와 hashCode()를 자동 생성한다.

주로 컬렉션 사용, 객체 비교, 엔티티 비교 등에 활용된다.

## 3.8 @Data

다음 기능들을 한 번에 제공하는 종합 어노테이션이다.

@Getter
@Setter
@ToString
@EqualsAndHashCode
@RequiredArgsConstructor
예시
```java
@Data
public class UserDto {
    private String name;
    private int age;
}
```
단, @Data는 편리하지만 엔티티 클래스에는 무분별하게 쓰지 않는 경우도 많다.

## 3.9 @Builder

빌더 패턴을 자동 생성한다.

예시
```java
@Builder
public class User {
    private String name;
    private int age;
}
```
사용 예:
```java
User user = User.builder()
        .name("EJ")
        .age(30)
        .build();
```        
## 4. Lombok 사용 방법
## 4.1 의존성 추가

프로젝트에 Lombok 라이브러리를 추가해야 한다.

### Maven 예시
```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```
### Gradle 예시

```java
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
```
## 4.2 IDE 플러그인 설치

Lombok은 컴파일 시점에 코드를 생성하므로, IDE에서 Lombok 플러그인이 없으면 코드가 정상이어도 빨간 줄이 뜰 수 있다.

따라서 IntelliJ에서는 다음이 필요하다.

Lombok 플러그인 설치
Annotation Processing 활성화
## 4.3 Annotation Processing 활성화

IntelliJ 기준:

Settings
Build, Execution, Deployment
Compiler
Annotation Processors
Enable annotation processing 체크

이 설정이 꺼져 있으면 Lombok이 제대로 동작하지 않을 수 있다.

## 5. Lombok 사용 예시
일반적인 서비스 클래스
```java
@Service
@RequiredArgsConstructor
public class AuthService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
}
```
설명:

@RequiredArgsConstructor가 생성자를 자동 생성한다.
final 필드를 기준으로 생성자 파라미터가 만들어진다.
직접 생성자를 작성하지 않아도 된다.
## 6. Lombok의 장점
## 6.1 코드량 감소

반복 코드 작성을 줄여준다.

## 6.2 핵심 로직 집중

보조 코드보다 실제 비즈니스 로직이 더 잘 보인다.

## 6.3 유지보수 편의

필드 추가/변경 시 관련 코드 작성량이 줄어든다.

## 7. Lombok 사용 시 주의점
## 7.1 코드가 눈에 안 보인다

실제로는 생성되지만 소스에 직접 안 보이기 때문에 초보자 입장에서는 헷갈릴 수 있다.

## 7.2 IDE 설정이 중요하다

플러그인 또는 annotation processing 설정이 안 되어 있으면 오류처럼 보일 수 있다.

## 7.3 @Data 남용 주의

편하다고 모든 클래스에 @Data를 붙이면 의도하지 않은 setter, equals/hashCode 등이 생성될 수 있다.

특히 엔티티에서는 신중하게 사용해야 한다.

## 8. 정리

Lombok은 자바의 반복 코드를 줄이기 위한 도구이다.

핵심 요약:

반복 코드 자동 생성
생산성 향상
가독성 개선
@Getter, @Setter, @RequiredArgsConstructor, @Builder 등이 대표 기능
IntelliJ에서는 플러그인 설치와 annotation processing 활성화가 필요

즉, Lombok은
**"자바의 귀찮은 반복 코드를 줄여주는 자동화 도구"**라고 이해하면 된다.