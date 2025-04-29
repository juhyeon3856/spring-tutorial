# 1차시 미션 (Spring Boot 튜토리얼)

## 1️⃣ spring-tutorial 완료

> 깃을 다뤄볼 수 있어서 좋았고,  
> 간단한 코드 작성과 테스트까지 해볼 수 있어 유익했습니다.

---

## 2️⃣ Spring이 지원하는 기술들 (IoC/DI, AOP, PSA 등) 조사

### 1. POJO란?

- 외부 기술(스프링 등)에 종속되지 않고, 순수한 자바 문법만으로 작성된 객체.
- 프레임워크 의존 없이 자유롭게 테스트하고 재사용할 수 있다.

### 2. IoC/DI (제어의 역전 / 의존성 주입)

- 객체 생성과 의존성 주입을 **스프링 컨테이너**가 대신 관리한다.
- 개발자는 비즈니스 로직에만 집중할 수 있다.
- 개발자가 직접 `new`로 객체를 만들지 않고, 스프링이 대신 주입하는 구조 → **제어의 역전(Inversion of Control)**

### 3. AOP (Aspect Oriented Programming)

- 공통 관심사(로깅, 트랜잭션, 보안 등)를 핵심 로직에서 분리해 코드 중복을 제거한다.
- 주로 서비스 계층이나 컨트롤러 계층 사이에서 동작하며, 필터처럼 요청을 가로채 처리할 수 있다.

### 4. PSA (Portable Service Abstraction)

- 외부 기술(JDBC, JMS 등)을 스프링이 일관된 방식으로 추상화한다.
- 기술이 바뀌어도 비즈니스 코드 수정 없이 그대로 사용할 수 있다.

---

## 3️⃣ Spring Bean이 무엇이고, Bean의 라이프사이클

- `@Configuration`이 붙은 클래스는 스프링 설정 클래스로 인식되어 애플리케이션이 구동된다.
- `@ComponentScan`을 통해 지정된 패키지에서 `@Component`, `@Service`, `@Repository`, `@Controller` 같은 어노테이션이 붙은 클래스를 자동으로 스캔하여 **빈(Bean)**으로 등록한다.

  > **(정리)** `@Component`는 "빈이 될 수 있는 후보"이고, `@ComponentScan`이 실제로 "찾아서 등록"한다.

- 등록된 빈은 기본적으로 **싱글톤 스코프**로 관리된다.

  - 애플리케이션 전체에서 하나의 인스턴스를 공유한다.
  - 필요에 따라 프로토타입 스코프 등 다른 스코프를 사용할 수도 있다.

- 빈의 생명주기 중:

  - `@PostConstruct` : 빈 초기화 직후 호출
  - `@PreDestroy` : 컨테이너 종료 직전에 호출

- 빈으로 등록된 객체는 `@Autowired`, 생성자 주입 등을 통해 스프링 컨테이너가 의존성을 주입한다.
- 개발자는 직접 객체를 `new`하지 않고, 스프링이 대신 관리하는 객체를 주입받아 비즈니스 로직에만 집중할 수 있다.

### 📌 어떤 걸 빈으로 등록하고, 어떤 건 직접 new하는가?

| 구분                     | 설명                                | 예시                                                                                                           |
| :----------------------- | :---------------------------------- | :------------------------------------------------------------------------------------------------------------- |
| 빈으로 등록하는 대상     | 비즈니스 로직, 공통 유틸성 컴포넌트 | `@Service`, `@Repository`, `@Controller`, `@Component`, 토큰 생성기, 암호화 유틸                               |
| 직접 new로 생성하는 대상 | 순수 데이터 객체나 임시 객체        | DTO, VO (`UserRequest`, `OrderResponse`), 단순 컬렉션, `LocalDateTime`, `ObjectMapper` 등 외부 라이브러리 객체 |

---

## 4️⃣ 스프링 어노테이션 심층 분석

### 어노테이션이란?

- 어노테이션은 "코드에 정보를 태그(Tag)처럼 다는 것"이다.
- 이 정보(태그)를 컴파일러나 스프링 같은 프레임워크가 읽고 해석해서 동작을 다르게 하도록 만드는 것이다.
- Java에서는 `@interface`로 선언하고, `@Retention`, `@Target` 같은 메타 어노테이션으로 사용 범위와 유지 정책을 설정한다.

### 커스텀 어노테이션 주요 사용 예시

1. AOP 포인트컷 대체
2. 실행 조건 제어 (Validation)
3. API 문서화/메타정보 전달
4. Bean 생성/스캐닝 힌트
5. 보안 권한 체크
6. 코드 가독성 향상

### 1. AOP 포인트컷 대체 예시

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MySecured {}
```

```java
@Aspect
@Component
public class SecurityAspect {

    @Pointcut("@annotation(com.example.MySecured)")
    public void securedMethod() {}

    @Before("securedMethod()")
    public void checkSecurity() {
        System.out.println("보안 체크 실행");
    }
}
```

### 2. 실행 조건 제어 (Validation) 예시

**PhoneNumberValidator.java**

```java
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class PhoneNumberValidator implements ConstraintValidator<PhoneNumber, String> {
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return false;
        }
        return value.matches("^\\d{10,11}$");
    }
}
```

**PhoneNumber.java**

```java
import jakarta.validation.Constraint;
import jakarta.validation.Payload;

@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneNumberValidator.class)
public @interface PhoneNumber {
    String message() default "전화번호 형식이 올바르지 않습니다.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

**UserRequest.java**

```java
public class UserRequest {
    @PhoneNumber
    private String phone;
}
```

**UserController.java**

```java
@RestController
public class UserController {

    @PostMapping("/user")
    public String createUser(@RequestBody @Valid UserRequest request) {
        return "성공!";
    }
}
```

---

---

## 5️⃣ 단위 테스트와 통합 테스트 탐구

| 구분        | 단위 테스트 (Unit Test)       | 통합 테스트 (Integration Test) |
| :---------- | :---------------------------- | :----------------------------- |
| 테스트 범위 | 메서드, 클래스 같은 작은 단위 | 여러 컴포넌트나 시스템 간 연결 |
| 실행 속도   | 매우 빠름                     | 상대적으로 느림                |
| 외부 의존성 | 없음 (Mocking)                | 실제 환경(DB, 서버 등) 사용    |
| 목적        | 기능 단독 검증                | 시스템 전체 흐름 점검          |
| 예시        | 서비스 메서드 로직 검증       | 회원가입 API 요청/응답 검증    |

✅  
**단위 테스트**는 빠르고 명확하게 작은 기능을 검증한다.  
**통합 테스트**는 시스템 전체가 유기적으로 작동하는지 점검한다.
