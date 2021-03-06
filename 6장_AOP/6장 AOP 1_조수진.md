### 6.1 트랜잭션 코드의 분리

비지니스 로직을 사이에 두고 트랜잭션 시작과 종료 로직이 위치

→ 분리 가능

→ 메서드 분리 → 클래스 분리 & DI 적용

⇒ 인터페이스 추출 & 프록시 패턴으로 트랜잭션 기능 분리 및 비지니스 로직 위임

![](/img/pic6-3.png)

→ 비지니스 로직의 코드를 작성할 때 트랜잭션에 대해 전혀 신경 쓸 필요가 없다

→ 비지니스 로직에 대한 테스트를 손쉽게 만들 수 있다

### 6.2 고립된 단위 테스트

테스트는 가능한 작아야 실패 원인을 찾기 쉽고 테스트의 의도나 내용이 분명해진다. 만들기도 쉽다.

→ 단위 테스트 먼저 생각할 것!

→ 복잡한 의존 관계를 가진 객체 테스트를 위해 테스트 대역으로 대상을 고립시킬 수 있다

- 스텁(상태 검증 - 리턴값 있는 경우 유용) vs 목(행위 검증)
- 단위 테스트 vs 통합 테스트
- Mockito 프레임워크

### 6.3 다이내믹 프록시와 팩토리 빈

- 프록시 패턴 vs 데코레이터 패턴은 같은 모양이지만, 사용하는 “목적”에 따라 분류
    - 데코레이터: 부가 기능을 런타임 시 다이나믹하게 부여하기 위해 사용
    - 프록시: 타깃에 대한 접근 방법을 제어하는 목적
- 기존 프록시를 만드는 것의 문제점
    - 매번 프록시 클래스 정의하고 인터페이스 구현하고, 메서드 작성하기 번거롭다
    - 부가기능 코드가 중복될 가능성이 많다

→ 리플렉션을 이용한 다이나믹 프록시 사용

![Untitled](/img/pic6-13.png)

- InvocationHandler에 부가 기능 작성 (`public Object invoke(Object proxy, Method method, Object[] args)` 메서드 가짐)

![Untitled](/img/pic6-14.png)

- 한 번에 여러 클래스에 공통적인 부가기능을 제공하는 것이 불가능
    
    (Handler가 target을 필드에 갖고 있기 때문)
    
- 한 타깃에 여러 부가기능을 적용하기 어렵다

### 6.4 스프링의 프록시 팩토리 빈

- ProxyFactoryBean: 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈
- 어드바이스(부가 기능)은 MethodInterceptor 인터페이스를 구현
    
    → 타깃 상관없이 독립적으로 만들 수 있음 → 싱글톤 빈 등록 가능
    
- 포인트컷: 부가기능 적용 대상 메소드 선정 방법
- 어드바이저 = 포인트컷 + 어드바이스

![Untitled](/img/pic6-18.png)

→ 템플릿/콜백 구조