# `필드주입, 생성자주입, final.md`

# RequiredArgsConstructor, @Autowired, 필드 주입, 생성자 주입, final 정리

## 1. 의존성 주입(DI)이란

의존성 주입(DI, Dependency Injection)이란  
**클래스가 필요로 하는 객체를 직접 생성하지 않고, 외부(Spring)가 넣어주는 방식**을 말한다.

예를 들어 `AuthController`가 `AuthService`를 필요로 한다고 하자.

직접 생성하는 방식:
```java
private AuthService authService = new AuthService();
```
주입받는 방식:
```java
private final AuthService authService;
```
그리고 생성자나 필드를 통해 Spring이 넣어준다.

즉,
**"내가 필요한 객체를 내가 만들지 않고 Spring이 주입해주는 것"**이 의존성 주입이다.

## 2. 필드 주입이란

필드 주입은 클래스의 필드에 직접 의존성을 넣는 방식이다.

예시
```java
@RestController
public class AuthController {

    @Autowired
    private AuthService authService;
}
```
설명:

authService 필드에 직접 의존성을 넣는다.
객체가 생성된 후 Spring이 필드에 값을 주입한다.

## 3. 생성자 주입이란

생성자 주입은 객체가 생성될 때 생성자를 통해 의존성을 받는 방식이다.

예시
```java
@RestController
public class AuthController {

    private final AuthService authService;

    public AuthController(AuthService authService) {
        this.authService = authService;
    }
}
```
설명:

AuthController가 생성될 때 AuthService를 함께 받는다.
받은 값을 필드에 저장한다.
객체가 만들어지는 시점에 필요한 의존성이 모두 확정된다.

## 4. 필드 주입과 생성자 주입의 차이
## 4.1 필드 주입
```java
@Autowired
private AuthService authService;
```
특징:

필드에 직접 주입
코드가 짧아 보임
예전 코드나 간단한 예제에서 자주 보임

단점:

이 클래스가 어떤 의존성을 가지는지 한눈에 덜 보임
final 사용이 어려움
테스트 시 직접 객체 생성이 불편함
구조가 상대적으로 덜 명확함
## 4.2 생성자 주입
```java
private final AuthService authService;

public AuthController(AuthService authService) {
    this.authService = authService;
}
```
특징:

객체 생성 시 의존성 주입
필요한 의존성이 명확하게 드러남
final 사용 가능
테스트가 쉬움
현재 스프링에서 더 선호되는 방식
## 5. @Autowired란

@Autowired는 Spring이 의존성을 주입하도록 지시하는 어노테이션이다.

사용 가능한 위치:

필드
생성자
setter 메서드
필드 주입 예시
```java
@Autowired
private AuthService authService;
```
생성자 주입 예시
```java
@Autowired
public AuthController(AuthService authService) {
    this.authService = authService;
}
```
다만 최근 스프링에서는 생성자가 하나뿐이면 @Autowired를 생략해도 자동으로 주입된다.

즉 아래도 정상 동작한다.
```java
public AuthController(AuthService authService) {
    this.authService = authService;
}
```
## 6. @RequiredArgsConstructor란

@RequiredArgsConstructor는 Lombok 어노테이션으로,
final 필드와 @NonNull 필드를 파라미터로 받는 생성자를 자동 생성한다.

예시
```java
@RequiredArgsConstructor
@RestController
public class AuthController {
    private final AuthService authService;
}
```
실제로는 다음 생성자가 자동으로 만들어진다.
```java
public AuthController(AuthService authService) {
    this.authService = authService;
}
```
즉,
@RequiredArgsConstructor는 생성자 주입 코드를 직접 작성하지 않아도 되게 해주는 도구이다.

## 7. @RequiredArgsConstructor와 @Autowired의 차이

둘은 비슷해 보여도 역할이 다르다.

## 7.1 @Autowired
Spring 어노테이션
의존성을 주입하는 역할
필드, 생성자, setter에 사용 가능
## 7.2 @RequiredArgsConstructor
Lombok 어노테이션
필요한 생성자를 자동 생성하는 역할
클래스에 사용

즉,

@Autowired = 주입을 지시
@RequiredArgsConstructor = 생성자를 자동 생성
## 8. 왜 @RequiredArgsConstructor를 많이 쓰는가

다음 조합이 자주 사용된다.
```java
@RequiredArgsConstructor
@Service
public class AuthService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
}
```
이 방식이 많이 쓰이는 이유:

생성자를 직접 안 써도 됨
생성자 주입 구조를 유지할 수 있음
코드가 짧고 깔끔함
의존성이 명확함

즉,
생성자 주입의 장점 + Lombok의 편의성을 동시에 가져갈 수 있다.

## 9. final이란

final은 한 번 값이 할당되면 다시 변경할 수 없도록 하는 키워드이다.

예시
```java
private final AuthService authService;
```
의미:

authService는 반드시 처음에 값이 들어가야 한다.
한 번 값이 정해지면 다시 바꿀 수 없다.
## 10. final을 사용하는 이유
## 10.1 필수 의존성임을 명확히 표현

컨트롤러나 서비스가 특정 객체 없이는 동작할 수 없다면, 이를 final로 표현할 수 있다.
```java
private final AuthService authService;
```
이 의미는 사실상 다음과 같다.

이 객체는 꼭 필요하다
생성 시점에 반드시 주입되어야 한다
중간에 바뀌면 안 된다
## 10.2 실수 방지

final이 없으면 나중에 실수로 값을 변경할 수 있다.
```java
this.authService = null;
```
또는
```java
this.authService = anotherService;
```
이런 식의 변경이 가능해진다.

final을 사용하면 이런 실수를 컴파일 단계에서 막을 수 있다.

## 10.3 객체 상태 안정화

객체가 생성될 때 필요한 값이 모두 정해지고, 이후 바뀌지 않기 때문에 구조가 더 안정적이다.

즉,
의존성이 고정되어 있어서 코드의 신뢰성이 높아진다.

## 11. 왜 생성자 주입과 final이 잘 어울리는가

생성자 주입은 객체를 만들 때 값을 넣는다.
```java
private final AuthService authService;

public AuthController(AuthService authService) {
    this.authService = authService;
}
```
이 구조는 다음과 같은 의미를 가진다.

필요한 의존성을 생성 시점에 받는다
한 번만 할당한다
이후 변경되지 않는다

즉,
생성자 주입과 final은 자연스럽게 함께 사용된다.

## 12. 필드 주입과 final

필드 주입은 이런 방식이다.
```java
@Autowired
private AuthService authService;
```
이 방식은 Spring이 객체 생성 후 필드에 주입하는 구조라서,
일반적으로 생성자 주입보다 final과의 조합이 덜 자연스럽다.

그래서 다음 조합이 더 권장된다.

private final
생성자 주입
@RequiredArgsConstructor

## 13. 실무에서 많이 쓰는 형태
컨트롤러
```java
@RequiredArgsConstructor
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    private final AuthService authService;
}
```
서비스
```java
@RequiredArgsConstructor
@Service
public class AuthService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
}
```
설명:

final로 필수 의존성 표현
@RequiredArgsConstructor로 생성자 자동 생성
Spring이 생성자를 통해 주입

이 조합이 가장 많이 쓰이는 편이다.

## 14. 한 번에 정리
## 14.1 필드 주입
```java
@Autowired
private AuthService authService;
```
의미:

필드에 직접 주입
짧아 보임
구조적으로는 생성자 주입보다 덜 권장
## 14.2 생성자 주입
```java
private final AuthService authService;

public AuthController(AuthService authService) {
    this.authService = authService;
}
```
의미:

생성 시점에 주입
의존성이 명확함
더 안정적임
## 14.3 @RequiredArgsConstructor
```java
@RequiredArgsConstructor
public class AuthController {
    private final AuthService authService;
}
```
의미:

필요한 생성자를 Lombok이 대신 생성
생성자 주입을 더 편하게 사용하게 해줌
## 14.4 final
```java
private final AuthService authService;
```
의미:

한 번 할당되면 바꿀 수 없음
필수 의존성 표현
실수 방지
객체 안정성 향상
## 15. 결론

핵심 요약:

의존성 주입은 필요한 객체를 Spring이 넣어주는 방식이다.
필드 주입은 필드에 직접 넣는 방식이다.
생성자 주입은 객체 생성 시 생성자를 통해 넣는 방식이다.
@Autowired는 Spring이 주입하도록 지시하는 어노테이션이다.
@RequiredArgsConstructor는 Lombok이 필요한 생성자를 자동 생성해주는 어노테이션이다.
final은 값을 한 번만 할당하도록 하여 실수를 막고 객체를 안정적으로 만든다.

실무에서 가장 많이 쓰는 조합은 보통 다음과 같다.
```java
@RequiredArgsConstructor
private final SomeService someService;
```
즉,
요즘 스프링에서는 final + 생성자 주입 + @RequiredArgsConstructor 조합을 가장 많이 사용한다고 이해하면 된다.