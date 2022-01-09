# 6. AOP

날짜: 2021년 12월 6일 → 2021년 12월 19일 속성: SR 주제: 6장 키워드: AOP, 관점 지향 프로그래밍

# AOP

![Untitled](images/001.jpeg)

## Intro

관점 지향 프로그래밍에 대한 내용을 이해해본다.

![Untitled](images/002.jpeg)

### 단위 테스트 (고립된 테스트)

> **기존에 작성했던 코드의 기능에 대해 AOP로 분리할 수 있는 부분을 확인**

- 테스트 대상의 의존 오브젝트를 파악하여 **`테스트 해야 하는 성격을 분리`**해본다.
- UserService의 기능이 동작하기 위해서 `**세 가지 기능**`이 필요하다.
    - `**데이터 접근**`에 필요한 로직인 UserDao
    - 서비스 레이어에서 DBMS의 `**트랜잭션의 관리**`를 위한 TransactionManager
    - **`메일 발송`**을 위한 MailSender

![Untitled](images/003.jpeg)

> **테스트 대상의 의존성을 최대한 줄이기 위한 목 오브젝트**

- MockMailSender라는 목 오브젝트를 통해 메일 발송 테스트하는 경우

![Untitled](images/004.jpeg)

- MockUserDao라는 목 오브젝트를 통해 데이터 수정 테스트 하는 경우

![Untitled](images/005.jpeg)

### 단위 테스트와 통합 테스트

단위 테스트와 통합 테스트를 작성하는 목적과 방식에 대해서 주의 깊게 살펴보아야 한다.

![Untitled](images/006.jpeg)

> **통합 테스트를 단위 테스트로 작성을 돕는 목 프레임워크**

- 두 가지 이상의 성격을 갖는 코드를 테스트 하기 위해서는 필히 통합 테스트로 작성해야 하나 목 프레임워크를 통해 단위 테스트로 작성할 수 있다.

![Untitled](images/007.jpeg)

### 프록시와 패턴

> **프록시와 사용 목적에 따른 패턴구분**


![Untitled](images/008.jpeg)


> **다이나믹 프록시**

- 기본적인 다이나믹 프록시의 동작방식

![Untitled](images/009.jpeg)

- 다이나믹 프록시 생성

![Untitled](images/010.jpeg)

- JDK 다이나믹 프록시와 스프링 ProxyFactoryBean의 구분

![Untitled](images/011.png)

### 6.6 트랜잭션 속성


### 6.7 어노테이션 트랜잭션 속성과 포인트컷

`포인트컷 표현식`과 `트랜잭션 속성`을 이용해 트랜잭션을 일괄적으로 적용하는 방식은 대부분의 상황에 적용할 수 있다.

보다 세밀하게 튜닝된 트랜잭션 속성을 적용해야 하는 경우에는 메서드 이름 패턴을 통해 일괄 적용하는 방식은 적합하지 않다.

스프링은 이러한 `문제점`을 개선하기 위해 `직접 타깃에 트랜잭션 속성정보를 갖는 어노테이션을 지정하는 방법`을 제공한다.

> **6.7.1 트랜잭션 어노테이션**

`타깃 부여할 수 있는 트랜잭션 어노테이션의 종류`에 대해서 알고 이해하기 전에 `메타 어노테이션`에 대해서 알고 있어야 한다.

- `@Transactional`

`@Transactional` 어노테이션의 타깃은 `메서드`와 `타입`이다.

따라서 `메서드`, `클래스`, `인터페이스`에 적용할 수 있으며, @Transactional 어노테이션을 트랜잭션 속성정보로 사용하도록 지정하면 모든 오브젝트를 자동으로 타깃 오브젝트로 인식한다.

이때 사용되는 포인트컷은 `TransactionAttributeSourcePointcut` 클래스로 표현식과 같은 선정기준을 갖지 않는다.

대신 @Transactional 어노테이션이 적용되어 있는 메서드, 클래스 레벨 상관없이 부여된 빈 오브젝트를 모두 찾아 포인트컷의 선정 결과로 돌려준다.

결국 `@Transactional 어노테이션`으로 `Transaction 속성을 정의하는 것`과 동시에 `포인트컷의 자동등록에도 사용되는 것`이다.

```java
import java.lang.annotation.*;

/**
 * 트랜잭션 속성의 모든 항목을 엘리먼트로 지정할 수 있으며, 디폴트 값이 설정되어 있어 있으므로 모두 생략이 가능하다.
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    String value() default "";
    Propagation propagation() default Propagation.REQUIRED;
    Isolation isolation() default Isolation.DEFAULT;
    int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
    boolean readOnly() default false;
    Class<? extends Throwable>[] rollbackFor() default {};
    String[] rollbackForClassName() default {};
    Class<? extends Throwable>[] noRollbackFor() default {};
    String[] noRollbackForClassName() default {};
}
```

- `트랜잭션 속성을 이용하는 포인트컷`

@Transactional 어노테이션을 적용할 때 어드바이저의 동작방식을 보면 TransactionInterceptor는 메서드 이름 패턴을 통해 부여되는 일괄적인 트랜잭션 속성정보 대신 @Transactional 어노테이션의 엘리먼트에서 트랜잭션 속성을 가져오는 AnnotationTransactionAttributeSource를 사용하게 된다.

이 방식은 포인트컷과 트랜잭션 속성을 어노테이션 하나로 지정할 수 있지만 트랜잭션 부가기능 적용 단위가 메서드 임을 생각하면

메서드 마다 @Transactional 어노테이션과 트랜잭션 속성을 설정해야 하는 중복코드를 생상하게 된다.

이를 개선하기 위해 스프링은 대체정책이라는 개념을 제공하여 중복코드를 줄일 수 있도록 돕는다.

![](images/012.png)

- `대체 정책`

스프링은 @Trnsactional을 적용할 때 4 단계의 대체정책을 이용하여 해준다.

메서드의 속성을 확인할 때 `타깃 메서드` -> `타킷 클래스` -> `선언 메서드` -> `선언 타입`의 순서에 따라 @Transactional이 적용되었는지 차례로 확인하고 가장 먼저 발견되는 속성정보를 사용하게 하는 방법이다.

`@Transactional`은 먼저 타입 레벨에 정의되고 `공통 속성`을 따르지 않는 메서드에 대해서만 메서드 레벨에 다시 `@Transactional`을 부여해주는 식으로 사용해야 한다.

기본적으로 @Transactional 적용 대상은 클라이언트가 사용하는 인터페이스에 정의한 메서드이므로 @Transactional도 타깃 클래스보다는 인터페이스에 두는게 바람직하다.

하지만 인터페이스를 사용하는 프록시 방식의 AOP가 아닌 방식으로 트랜잭션을 적용하면 인터페이스에 정의한 @Transactional은 무시되기 때문에 안전하게 타깃 클래스에 @Transactional을 두는 방법을 `권장`한다.

프록시 방식 AOP의 종류와 특징, 또는 비 프록시 방식 AOP의 동작 원리를 잘 이해하고 있고 그에따라 @Transactional의 적용 대상을 적절하게 변경해줄 확신이 있거나, 반드시 인터페이스를 사용하는 타깃에만 트랜잭션을 적용하겠다는 확신이 있다면 인터페이스에 @Transactional을 적용하고, 그게 아니라면 타깃 클래스와 타깃 메서드에 적용하는 편이 좋다.

- `트랜잭션 어노테이션 사용을 위한 설정`

@Transactional을 이용한 트랜잭션 속성을 사용하는데 필요한 설정은 어드바이저, 어드바이스, 포인트컷, 어노테이션을 이용하는 트랜잭션 속성정보가 한번에 등록된다.

xml 설정의 경우 `<tx:annotation-driven />` Java Config의 경우 `@EnableTransactionManagement`를 사용한다.

> **6.7.2 트랜잭션 어노테이션 적용**

어노테이션에 대한 대체 정책의 순서는 타킷 클래스가 인터페이스보다 우선하므로 모든 메서드의 트랜잭션은 디폴트 속성을 갖게 된다.


```java
@Transactional // 인터페이스 레벨에 디폴트 속성으로 적용
public interface UserService {
    void add(User user);
    void deleteAll();
    void update(User user);
    void upgradeLevels();
    
    @Transactional(readOnly = true) // 읽기 전용 속성의 트랜잭션이 필요한 경우 메서드 레벨에 적용
    User get(String id);

    @Transactional(readOnly = true)
    List<User> getAll();
}
```

```java
@Transactiional // 클래스 레벨에 트랜잭션 디폴트 속성으로 적용
public class UserServiceImpl implements UserService {
    // ...
}
```

위 트랜잭션 적용에 대해서 필히 테스트 코드를 작성하여 트랜잭션의 대체정책이 기대한 대로 동작하는지 확인해야 한다.

예를 들어 인터페이스에는 getAll() 메서드를 읽기전용 속성을 갖고 있으나 타깃 클래스에 디폴트 속성을 갖고 있는 점을 확인하기 위해

읽기 전용 속성을 검증하는 테스트 코드를 작성해 보는 것이다.

### 6.8 트랜잭션 지원 테스트

> **6.8.1 선언적 트랜잭션과 트랜잭션 전파 속성**

`트랜잭션 전파 속성`은 `REQUIRED`를 기본 속성으로 사용하고 있고 이는 앞서 진행 중인 트랜잭션이 있으면 참여하고, 없으면 자동으로 새로운 트랜잭션을 시작한다.

일반적으로는 `DB 트랜잭션`은 `단위 업무`와 일치하도록 해야 한다. 하지만 `단위 업무`가 `작업 단위가 다른 비즈니스 로직으로 구성`될 수 있다.

그런 경우 앞선 작업이 정상적으로 수행되고, 뒤 이어 후속 작업이 어떠한 이유 인하여 실패하는 경우 앞선 작업까지 롤백이 되어야 한다.

이것이 가능하도록 돕는 것이 `트랜잭션 전파`라는 기법으로 연속적인 작성된 비즈니스 로직에 대한 트랜잭션의 범위를 확장할 수 있다.

`트랜잭션 전파 기법`이 없는 경우, DB 트랜잭션을 어플리케이션 레벨에서 여러 비즈니스 로직에 대한 트랜잭션을 관리할 수 없기 때문에 결국에는 DB 쿼리에 하나 이상의 비즈니스 로직을 담게 될 것이다. 이는 코드의 중복, 유지보수에 대한 부분에서 효율적이지 못한 결과를 낼 수 있다.

스프링에서는 트랜잭션을 사용할 수 있도록 지원하는 방법이 2가지가 있는데 첫 번째는 `선언적 트랜잭션(declarative transaction)`과 두 번째는 `프로그램에 의한 트랜잭션(programmatic transaction)`을 제공하며 선언적 방식의 트랜잭션을 사용하는 것이 바람직하다.

- 선언적 트랜잭션
  - AOP를 이용해 코드 외부에서 트랜잭션의 기능을 부여해주고 속성을 지정할 수 있도록 제공한다.
- 프로그램에 의한 트랜잭션
  - `TransactionTemplate`이나 개별 데이터 기술의 트랜잭션 API를 사용해 직접 코드에서 사용하는 방법이다.


> **6.8.2 트랜잭션 동기화와 테스트**

트랜잭션의 자유로운 `전파`와 그로 인한 유연한 개발이 가능할 수 있었던 배경에 AOP가 있다.

한 가지 중요한 기술적인 기반은 `스프링의 트랜잭션 추상화`이다.

트랜잭션 기술에 상관없이 DAO에서 일어나는 작업들을 하나의 트랜잭션으로 묶어서 추상 레벨에서 관리하게 해주는 트랜잭션 추상화로 인해

비로소 선언적 트랜잭션이나 트랜잭션 전파 등이 가능해진 것이다.

- `트랜잭션 매니저와 트랜잭션 동기화`

트랜잭션 추상화 기술의 핵심은 `트랜잭션 매니저`와 `트랜잭션 동기화`이다.

스프링는 `PlatformTransactionManager 인터페이스`를 구현한 트랜잭션 매니저를 통해 구체적인 트랜잭션 기술의 종류에 상관없이 일관된 트랜잭션 제어를 가능하게 한다.

`트랜잭션의 동기화 기술`은 진행중인 트랜잭션이 있는지 확인하고, `트랜잭션 전파 속성`에 따라서 `트랜잭션 전파 기능`의 속성 별 동작하게 한다.

`트랜잭션의 전파 속성`은 `REQUIRED`, `SUPPORTS`, `MANDATORY`, `REQUIRES_NEW`, `NOT_SUPPORTED`, `NEVER`, `NESTED`등이 있으며 기본적인 트랜잭션의 전파 속성은 REQUIRED이다.

일반적으로는 AOP를 통해 트랜잭션의 전파 속성에 따라 동기화할 수도 있지만, 특별한 경우 TransactionManager를 통해 트랜잭션을 제어할 수도 있다.

스프링 테스트 컨텍스트를 사용하는 경우 PlatformTransactionManager Bean을 가져와 핸들링할 수도 있다.

```java
public class UserServiceTest {
    @Autowired
    private PlatformTransactionManager platformTransactionManager;
    @Autowired
    private UserService userService;
    
    @Test
    public void transactionSync() {
        userService.deleteAll();
        
        userService.add(users.get(0));
        userService.add(users.get(1));
    }
}
```

위 테스트 메서드가 실행 되는 동안 몇 개의 트랜잭션이 만들어 졌을까? UserService의 모든 메서드에는 트랜잭션이 적용되었다면 총 3개의 트랜잭션이 만들어졌을 것이다.

모두 독립적인 트랜잭션 안에서 실행이 되었을 것이다. 왜냐하면 기본적인 트랜잭션 전파 속성이 REQUIRED 이기 때문에 새로운 트랜잭션으로 각각 수행되었다.

- `트랜잭션 매니저를 이용한 테스트용 트랜잭션 제어`

여기서 시도해 볼 내용은 각각의 트랜잭션을 하나의 트랜잭션으로 동작하게 하는 것이다.

결국 트랜잭션의 전파 속성이 REQUIRED라는 속성을 이용하여 3개의 트랜잭션을 묶기 위해 메서드 호출 전 트랜잭션 시작되도록 한다는 것이다.

테스트 코드로 `트랜잭션 매니저`를 이용해 `트랜잭션을 시작시키고 이를 동기화` 하도록 한다.

```java
public class UserServiceTest {
    @Autowired
    private PlatformTransactionManager platformTransactionManager;
    @Autowired
    private UserService userService;

    @Test
    public void transactionSync() {
        DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
        TransactionStatus txStatus = platformTransactionManager.getTransaction(txDefinition);
        userService.deleteAll();
  
        userService.add(users.get(0));
        userService.add(users.get(1));
        
        platformTransactionManager.commit(txStatus);
    }
}
```

위 코드는 트랜잭션 매니저를 이용해 트랜잭션을 미리 시작하게 만드는 테스트이다.

- `트랜잭션 동기화 검증`

트랜잭션 매니저를 통해 코드를 구현 한 것까지는 좋지만 이에 대해 검증하는 코드는 필수이다.

정말 트랜잭션 매니저를 통해 세 개의 메서드의 트랜잭션이 동기화 되었는지 확인하는 작업이 필요하다.

이를 테스트 하는 방법은 트랜잭션의 속성에서 읽기전용이라는 속성을 통해 검증하는 것이다.

즉, 쓰기 가능의 트랜잭션 속성을 갖는 deleteAll() 전에 앞서 시작된 트랜잭션 상태를 읽기전용으로 하여 진행해보도록 한다.

읽기 전용 트랜잭션이 시작되고나서, 쓰기 작업이 수행되면 `TransientDataAccessResourceException` 이라는 예외가 발생하게 되는 것을 이용하여 트랜잭션 동기화를 검증한다.

`TransientDataAccessResourceException`는 읽기전용 트랜잭션에 대해 쓰기 작업을 했을때 발생하는 예외이다.

```java
public class UserServiceTest {
    @Autowired
    private PlatformTransactionManager platformTransactionManager;
    @Autowired
    private UserService userService;

    @Test
    public void transactionSync() {
        DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
        txDefinition.setReadOnly(true); // 읽기전용 트랜잭션으로 정의
        TransactionStatus txStatus = platformTransactionManager.getTransaction(txDefinition);
        
        userService.deleteAll(); // 읽기전용 속성을 위반하여 예외가 발생하는 라인
  
        userService.add(users.get(0));
        userService.add(users.get(1));
        
        platformTransactionManager.commit(txStatus);
    }
}
```

예외가 발생하는 것을 확인함으로써 테스트 코드 내에 시작한 트랜잭션에 deleteAll() 메서드가 참여하고 있다는 확신을 얻을 수 있다.

`스프링의 트랜잭션 추상화`가 제공하는 `트랜잭션 동기화 기술`과 `트랜잭션 전파 속성` 덕분에 테스트도 트랜잭션으로 묶을 수 있다.

이러한 방식은 선언적 트랜잭션이 적용된 서비스 메서드에만 적용되는 것이 아니라 JdbcTemplate과 같이 스프링이 제공하는 데이터 엑세스 추상화를 적용한 DAO에도 동일하게 사용 가능하다.

```java
public class UserServiceTest {
    @Autowired
    private PlatformTransactionManager platformTransactionManager;
    @Autowired
    private UserService userService;
  
    @Test
    public void transactionSync() {
      DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
      txDefinition.setReadOnly(true); // 읽기전용 트랜잭션으로 정의
      TransactionStatus txStatus = platformTransactionManager.getTransaction(txDefinition);
  
      userDao.deleteAll(); // JdbcTemplate을 사용하더라도 이미 시작된 트랜잭션이 있다면 자동으로 참여하게 되어 예외가 발생한다.
  
      // ...
    }
}
```

트랜잭션의 롤백 테스트도 수행되는지 확인해보는 코드를 작성한다.

```java
public class UserServiceTest {
    @Autowired
    private PlatformTransactionManager platformTransactionManager;
    @Autowired
    private UserService userService;
  
    @Test
    public void transactionSync() {
      userService.deleteAll();
      assertThat(userDao.getCount()).isOne(); // 트랜잭션 시작 전 초기화
      
      DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
      TransactionStatus txStatus = platformTransactionManager.getTransaction(txDefinition);
      
      userService.add(users.get(0));
      userService.add(users.get(1));
      assertThat(userDao.getCount()).isEqualTo(2); // userDao.getCount()를 통해 두개의 데이터가 들어갔는지 확인
      
      platformTransactionManager.rollback(txStatus);
      
      assertThat(userDao.getCount()).isZero(); // 롤백 후 트랜잭션 시작 이전의 상태임을 확인
    }
}
```

지금까지 작업을 통해 테스트 안에서 트랜잭션을 조작할 수 있는 방법을 이해했다면 올바르게 이해한 것이다.

이 방식을 통해 `ORM에서 세션에서 분리된(detached) 엔티티의 동작`을 확인할 때도 유용하다.

테스트 메서드 안에서 `트랜잭션을 여러 번 만들 수`도 있고, `트랜잭션의 속성에 따라서 여러 메서드를 조합해 사용`할 때 어떠한 결과가 나오는지 미리 검증도 가능하다.

- 롤백 테스트

롤백 테스트의 목표는 테스트 내에 모든 DB 작업을 하나의 트랜잭션 안에서 동작하게 하고 테스트가 끝나면 무조건 롤백하는 것이다.

```java
public class UserServiceTest {
    @Autowired
    private PlatformTransactionManager platformTransactionManager;
    @Autowired
    private UserService userService;
  
    @Test
    public void transactionSync() {
        DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
        TransactionStatus txStatus = platformTransactionManager.getTransaction(txDefinition);
        
        try {
            userService.deleteAll();
            userService.add(users.get(0));
            userService.add(users.get(1));
            assertThat(userDao.getCount()).isEqualTo(2); // userDao.getCount()를 통해 두개의 데이터가 들어갔는지 확인
        } finally {
            platformTransactionManager.rollback(txStatus);
            assertThat(userDao.getCount()).isZero(); // 롤백 후 트랜잭션 시작 이전의 상태임을 확인
        }
    }
}
```

테스트를 작성할 때는 대상 메서드 외에 DB를 사용하게 되는 경우 DB의 데이터와 상태가 중요한데, 더 큰 문제는 DB의 데이터가 바뀌는 것이 가장 문제이다.

테스트가 어떤 순서로 동작할 지 알 수 없고, 성공 했을 대와 실패 했을 때 DB에 다른 방식으로 영향을 줄 수 있기 때문에, 테스트 할 때 마다 DB 데이터를 초기화 하는 것이 중요하다.

결국 롤백 테스트는 테스트를 진행하는 동안에 조작한 데이터를 모두 롤백하고 테스트를 시작하기 전 상태로 만들어 주기 때문에 테스트에 대한 성공, 실패, 예외 모두 상관이 없다.

테스트에서 트랜잭션을 제어할 수 있기 때문에 롤백 테스트가 가능한 것이다.

> **6.8.3 테스트를 위한 트랜잭션 애노테이션**

@Transactional 어노테이션은 타깃 클래스 또는 인터페이스에 부여하여 트랜잭션을 적용할 수 있는데 이는 테스트 코드를 작성하는데도 동일하게 사용할 수 있다.

스프링의 컨텍스트 테스트 프레임워크에서 제공하는 `@ContextConfiguration`을 클래스에 부여하여 스프링 컨테이너를 초기화하고, 필요한 빈만을 등록하여 테스트에 필요한 빈을 `@Autowired`를 통해 자유롭게 접근할 수 있다.

- `@Transactional`

어노테이션을 적용한 메서드에 트랜잭션 경계가 자동으로 설정되어 트랜잭션 관련 작업을 하나로 묶을 수 있다.

```java
public class UserServiceTest {
    @Autowired
    private UserService userService;
  
    @Test
    @Transactional
    public void transactionSync() {
        userService.deleteAll();
        userService.add(users.get(0));
        userService.add(users.get(1));
    }
}
```

트랜잭션의 적용 여부를 확인하는 작업이 필요한 경우 트랜잭션의 속성을 읽기 전용으로 설정하여 동일한 테스트를 진행해본다.

```java
public class UserServiceTest {
    @Autowired
    private UserService userService;
  
    @Test
    @Transactional(readOnly = true)
    public void transactionSync() {
        userService.deleteAll(); // 읽기 전용 트랜잭션에서 쓰기 요청을 하는 경우 예외 발생
        userService.add(users.get(0));
        userService.add(users.get(1));
    }
}
```

- `@Rollback`

테스트용 트랜잭션은 테스트가 끝나면 자동으로 롤백이 된다.

혹시나 DB에 반영하고 싶은 경우 @Rollback 어노테이션을 사용하여 속성을 false 로 갖게 하면 트랜잭션이 모두 종료되더라도 DB가 롤백 되지 않는다.

다만 테스트 데이터가 DB에 데이터를 넣는건 신중하게 생각해야 한다. 물론 테스트 DB가 따로 있는 경우, 개발자들 간의 서로 예약된 내용이 있는 경우 사용하도록 한다.

```java
public class UserServiceTest {
    @Autowired
    private UserService userService;
  
    @Test
    @Transactional
    @Rollback(false)
    public void transactionSync() {
        userService.deleteAll();
        userService.add(users.get(0));
        userService.add(users.get(1));
    }
}
```

- `@TransactionConfiguration`

@Transactional 어노테이션은 테스트 클래스에 적용하여 모든 메서드에 일괄 적용할 수 있고, @Rollback 어노테이션은 메서드 레벨에만 적용이 가능하다.

테스트 클래스의 모든 메서드에 트랜잭션을 적용하면서 모든 트랜잭션이 롤백되지 않고 커밋되게 하려면 @TransactionConfiguration 어노테이션을 이용하여 롤백에 대한 공통 속성을 지정할 수 있다.

```java
@Transactional
@TransactionConfiuguration(defaultRollback = false)
public class UserServiceTest {
    @Autowired
    private UserService userService;
  
    @Test
    @Transactional
    @Rollback(false)
    public void transactionSync() {
        userService.deleteAll();
        userService.add(users.get(0));
        userService.add(users.get(1));
    }
}
```

- `NotTransactional과 Propagation.NEVER`

테스트 클래스 안에서 일부 메서드에만 트랜잭션이 필요한 경우 메서드 레벨에 @Transactional을 적용한다.

또는 대부분 메서드에 트랜잭션이 필요한 경우 클래스 레벨에 @Transactional을 적용한다.

굳이 트랜잭션이 필요 없는 메서드도 굳이 적용해야 할까? 해당 메서드에만 테스트 메서드에 의한 트랜잭션이 시작되지 않도록 트랜잭션의 전파 속성을 변경한다.

@Transactional(propagation = Propagation.NEVER)는 트랜잭션이 시작되지 않도록 하는 전파 속성으로 위의 목적을 달성할 수 있다.

- `효과적인 DB 테스트`

테스트 내에서 트랜잭션을 제어할 수 있는 네 가지 어노테이션을 잘 활용하게 되면 DB가 사용되는 통합 테스트를 만들 때 편리하다.

일반적으로 의존, 협력 오브젝트를 사용하지 않고 고립된 상태에서 테스트를 진행하는 `단위 테스트`, DB 같은 외부의 리소스나 여러 계층의 클래스가 참여하는 `통합 테스트`는 아예 클래스를 구분해서 따로 만드는 것이 좋다.

통합 테스트와 단위 테스트를 구분하게 되면 보다 효율적으로 어노테이션을 활용할 수 있다.

테스트는 어떤 경우에도 서로 의존하면 안된다. 테스트가 진행되는 순서나 앞의 테스트의 성공 여부가 다음 테스트에 영향을 주는 것도 허용하지 않는다.

어떠한 순서로 진행되더라도 일정한 결과를 내야 한다.


### 6.9 정리

> 책 읽은 척하기

**트랜잭션 경계설정 기능**을 성격이 다른 **비즈니스 로직 클래스**에서 분리하여 적용할 수 있는 방법을 찾아 보면서 최종적으로는 모듈화할 수 있는 AOP 기술을 통해 해결하였다.

> 6장에서 무엇을 했고 어떤 부분을 정리했는지 살펴보기

- **트랜잭션 경계설정 코드**를 분리해서 별도의 클래스를 만들고 **비즈니스 로직 클래스**와 동일한 인터페이스를 구현하면 DI의 확장기능을 이용해 클라이언트의 변경없이 깔끔하게 분리된 트랜잭션 **부가기능**을
  만들 수 있다.
    - `데코레이터 패턴` 및 `프록시 패턴`을 통해 트랜잭션 기능을 적용하는 방식으로 앞에서 설명되었다.
- 트랜잭션처럼 환경과 외부 리소스에 영향을 받는 코드를 분리하면 `비즈니스 로직에만 충실한 테스트`를 만들 수 있다.
- **목 오브젝트**를 활용하면 의존관계 속에 있는 오브젝트도 손쉽게 **고립된 테스트**로 만들 수 있다.
- `DI를 이용한 트랜잭션의 분리`는 `데코레이터 패턴`과 `프록시 패턴`으로 이해될 수 있다.
- 번거로운 프록시 클래스 작성은 **JDK의 다이내믹 프록시**를 사용하면 간단하게 만들 수 있다.
- **다이내믹 프록시**는 **스태틱 팩토리 메소드**를 사용하기 때문에 빈으로 등록하기 번거롭다. 따라서 **팩토리 빈**으로 만들어야 한다. 스프링은 자동 프록시 생성 기술에 대한 추상화 서비스를 제공하는 **
  프록시 팩토리 빈(ProxyFactoryBean)**을 제공한다.
- **프록시 팩토리 빈**의 설정이 반복되는 문제를 해결하기 위해 **자동 프록시 생성기**와 **포인트컷** 을 활용할 수 있다. 자동 프록시 생성기는 부가기능이 담긴 어드바이스를 제공하는 프록시를 스프링
  컨테이너 초기화 시점에 자동으로 만들어준다.
- 포인트컷은 AspectJ 포인트컷 표현식을 사용해서 작성하면 펀리하다.
- **AOP**는 **OOP**만으로는 모듈화하기 힘든 부가기능을 효과적으로 모듈화하도록 도와주는 기 술이다.
- 스프링은 자주 사용되는 AOP 설정과 **트랜잭션** 속성을 지정하는 데 사용할 수 있는 **전용 태 그**를 제공한다.
- **AOP**를 이용해 **트랜잭션 속성을 지정하는 방법**에는 포인트컷 표현식과 메소드 이름 패턴 을 이용하는 방법과 타깃에 직접 부여하는 @Transactional 애노테이션을 사용하는 방법이 있다.
- **@Transactional**을 이용한 트랜잭션 속성을 테스트에 적용하면 손쉽게 DB를 사용하는 코드의 테스트를 만들 수 있다.
