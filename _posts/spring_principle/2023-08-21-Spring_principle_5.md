---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(싱글톤 컨테이너)"
categories: spring
author_profile: false
typora-root-url: ../






---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[인프런 김영한 님의 스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



-----------

이번 시간에는 싱글톤 컨테이너에 대해 알아보자.



## 웹 애플리케이션과 싱글톤

- 스프링은 태생이 기업용 온라인 서비스 기술을 지원하기 위해 탄생했다.
- 대부분의 스프링 애플리케이션은 웹 애플리케이션이다.
- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.
- ![image-20230822163837895](/images/2023-08-21-Spring_principle_4/image-20230822163837895.png)
- 우리가 만들었던 스프링 없는 순수한 DI컨테이너인 `AppConfig`는 요청을 할 때 마다 객체를 새로 생성한다.
- 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸되기 때문에 메모리 낭비가 심하다.
- 해결방안은 해당 객체가 딱 1개만 생성되고, 공유하도록 설계하면 된다. -> **싱글톤 패턴**

<br>



## 싱글톤 패턴

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
- 그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.
  - private생성자를 사용해서 외부에서 임의로 new키워드를 사용하지 못하도록 막아야 한다.

<br>

`hello.singleton.SingletonService`

```java
public class SingletonService {

    // static 영역에 객체를 딱 1개만 생성해둔다
    private static final SingletonService instance = new SingletonService();

    // public으로 열어서 객체 인스턴스가 필요하면 이 메서드를 통해서만 조회하도록 허용한다.
    public static SingletonService getInstance() {
        return instance;
    }
    
    // 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다.
    private SingletonService() {
    }

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```



<br>

싱글톤 패턴이 잘 적용되었는지 테스트 코드를 통해 확인해보자. 

```java
@Test
@DisplayName("싱글톤 패턴을 적용한 객체 사용")
void singletonServiceTest() {
    SingletonService singletonService1 = SingletonService.getInstance();
    SingletonService singletonService2 = SingletonService.getInstance();

    System.out.println("singletonService1 = " + singletonService1);
    System.out.println("singletonService2 = " + singletonService2);

    assertThat(singletonService1).isSameAs(singletonService2);
}

```

<img src="/images/2023-08-21-Spring_principle_4/image-20230822164619663.png" alt="image-20230822164619663" style="zoom:67%;" />

- 호출할때 마다 같은 객체 인스턴스를 반환하는 것을 확인할 수 있다. 



<br>

싱글톤 패턴을 적용하면 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 사용할 수 있다. 하지만 싱글톤 패턴은 수 많은 문제점들을 가지고 있다.

### 싱글톤 패턴 문제점

- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다. 
- 의존관계상 클라이언트가 구체 클래스에 의존한다. -> DIP를 위반한다.
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 결론적으로 유연성이 떨어진다.

<br>

## 싱글톤 컨테이너

스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다. 지금까지 학습한 스프링 빈이 바로 싱글톤으로 관리되는 빈이다. 

- 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도 객체 인스턴스를 싱글톤으로 관리한다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글톤 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
  - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  - DIP, OCP, 테스트, private 생성자로부터 자유롭게 싱글톤을 사용할 수 있다. 

<br>

`스프링 컨테이너를 사용하는 테스트 코드`

```java
@Test
@DisplayName("스프링 컨테이너와 싱글톤")
void springContainer() {

//    AppConfig appConfig = new AppConfig();

     ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

     MemberService memberService1 = ac.getBean("memberService", MemberService.class);
     MemberService memberService2 = ac.getBean("memberService", MemberService.class);

     // 참조값이 다른지 확인
     System.out.println("memberService1 = " + memberService1);
     System.out.println("memberService2 = " + memberService2);

     assertThat(memberService1).isSameAs(memberService2);
    }
}
```

`테스트 실행 결과`

<img src="/images/2023-08-21-Spring_principle_4/image-20230822172621270.png" alt="image-20230822172621270" style="zoom:67%;" />

<br>



### 싱글톤 컨테이너 적용 후

<img src="/images/2023-08-21-Spring_principle_4/image-20230822172725502.png" alt="image-20230822172725502" style="zoom:80%;" />

- 스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다. 



<br>

## 싱글톤 방식의 주의점

- 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.
- **무상태**(stateless)로 설계해야 한다.
  - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
  - 가급적 읽기만 가능해야 한다.
  - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다.

<br>



## @Configuration과 싱글톤

`AppConfig`코드를 보자.

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy(); 
    }

}
```

- memberService빈을 만드는 코드를 보면 `memberRepository()`를 호출한다.
  - 이 메서드를 호출하면 `new MemoryMemberRepository()` 를 호출한다.
- orderService 빈을 만드는 코드도 동일하게 `memberRepository()` 를 호출한다.
  - 이 메서드를 호출하면 `new MemoryMemberRepository()` 를 호출한다. 

결과적으로 각각 다른 2개의 `MemoryMemberRepository` 가 생성되면서 싱글톤이 깨지는 것 처럼 보인다. 그래서 `AppConfig`에 호출 로그를 남겨보았다.

<br>

`AppConfig`에 호출 로그 남기기

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        System.out.println("call AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

<br>

호출 로그를 남기고 출력을 해보았더니 모두 1번씩만 호출되었다. 어떻게 이렇게 되는 것일까 ?

```java
call AppConfig.memberService
call AppConfig.memberRepository
call AppConfig.orderService
```

<br>

## @Configuration과 바이트코드 조작의 마법

스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용한다. 모든 비밀은 @Configuration 을 적용한 AppConfig 에 있다.

<br>

다음 코드를 보자.

```java
@Test
void configurationDeep() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
  
  // AppConfig도 스프링 빈으로 등록된다.
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
  // 출력 bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$5bbd9900
}
```

- 사실 `AnnotationConfigApplicationContext` 에 파라미터로 넘긴 값은 스프링 빈으로 등록된다. 그래서 `AppConfig` 도 스프링 빈이 된다.
- `AppConfig` 스프링 빈을 조회해서 클래스 정보를 출력해보자.
- <img src="/images/2023-08-21-Spring_principle_4/capture 54.png" alt="capture 54" style="zoom:80%;" />

<br>

순수한 클래스라면 `class hello.core.AppConfig`의 형태로 출력되어야 한다. 하지만 클래스 명에 ~~CGLIB가 붙으며 상당히 복잡해진 것을 볼 수 있다. 이것은 내가 만든 클래스가 아니라 스프링이 **CGLIB라는 바이트코드 조작 라이브러리**를 사용해서 `AppConfig`클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것이다. 

<img src="/images/2023-08-21-Spring_principle_4/image-20230822174347137.png" alt="image-20230822174347137" style="zoom:80%;" />

<br>

그 임의의 다른 클래스가 싱글톤이 보장되도록 해준다. 바이트코드를 조작하여 @Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다. 덕분에 싱글톤이 보장되는 것이다.

<br>

만약 코드에 `Configuration`을 적용하지 않고 `@Bean`만 적용한다면 스프링 빈으로 등록은 되지만, 싱글톤은 보장되지 않는다.

그러므로 스프링 설정 정보는 항상 `Configuration`을 사용하자. 





















