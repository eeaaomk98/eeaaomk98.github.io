---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(객체지향 원리 적용)"
categories: spring
author_profile: false
typora-root-url: ../




---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



저번 게시글에서 할인 정책을 적용하여 주문하기까지 진행해 보았다.

이번에는 고정 금액 할인이 아닌 정률% 할인을 적용하여 금액당 10%의 할인을 적용하는 새로운 정률 할인 정책을 추가해보자.



<br>

`hello.core.discount.RateDiscountPolicy` 

```java
public class RateDiscountPolicy implements DiscountPolicy{

    private int discountPercent = 10;

    // 등급이 DIAMOND이면 금액에서 10%를 할인
    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.DIAMOND) {
            return price * discountPercent / 100;
        } else {
            return 0;
        }
    }
}
```



<br>

추가한 할인 정책을 애플리케이션에 적용해보자. 할인 정책을 변경하려면 클라이언트인 `OrderServiceImpl`코드를 수정해야 한다.

`OrderServiceImpl`

```java
public class OrderServiceImpl implements OrderService {
  //    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
      private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
  }
```

하지만 이 코드에는 문제점이 있다. 

<br>

![image-20230810144201884](/images/2023-08-10-Spring_principle_3/image-20230810144201884.png)

우리는 역할과 구현을 분리하여 인터페이스와 구현 객체를 분리하여 코드를 작성했다.

하지만 주문서비스 클라이언트인 `OrderServiceImpl`은 `DiscountPolicy`인터페이스에 의존하면서 구현(구체) 클래스인 `RateDiscountPolicy`에도 의존을 하고 있는 상태이다. 따라서 OCP와 DIP를 위반하고 있는 것이다.

<br>

그래서 이 문제를 해결하기 위해서 **인터페이스에만 의존하도록** 의존관계를 변경해보자. 



`OrderServiceImpl`

```java
public class OrderServiceImpl implements OrderService{

    // private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
    private final DiscountPolicy discountPolicy;
}
```

`OrderServiceImpl`이 인터페이스에만 의존하도록 해주었으나 구현체가 없어서 코드를 실행할 수 없다. 

이 문제를 해결하려면 누군가 클라이언트인 `OrderServiceImpl`에 `DiscountPolicy`의 구현 객체를 대신 생성하고 주입해주어야 한다. 



<br>

## AppConfig 등장

애플리케이션의 전체 동작 방식을 구성(config)하기 위해, **구현 객체를 생성**하고, **연결**하는 책임을 가지는 별도의 설정 클래스를 생성한다. 

`hello.core.AppConfig`

```java
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    public static MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy() {
//        return new FixDiscountPolicy();
        return new RateDiscountPolicy();  // 정률 할인 정책으로 변경
    }

}
```

`AppConfig`는 애플리케이션의 실제 동작에 필요한 **구현 객체를 생성**한다. 

- MemberServiceImpl
- MemoryMemberRepository
- OrderServiceImpl
- FixDiscountPolicy

<br>

`AppConfig`는 생성한 객체 인스턴스의 참조를 **생성자를 통해서 주입(연결)**해준다.

- MemberServiceImpl -> MemoryMemberRepository
- OrderServiceImpl -> MemoryMemberRepository, FixDiscountPolicy



<br>

이제 각 클래스에 생성자를 만들어주자.



`MemberServiceImpl` - 생성자 주입

```java
public class MemberServiceImpl implements MemberService {

     private final MemberRepository memberRepository;
        
     public MemberServiceImpl(MemberRepository memberRepository) {
         this.memberRepository = memberRepository;
     }

     public void join(Member member) {
         memberRepository.save(member);
     }

     public Member findMember(Long memberId) {
         return memberRepository.findById(memberId);
     } 
}

```



<br>

`OrderServiceImpl` - 생성자 주입

```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;

public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```

설계 변경으로 `MemberServiceImpl`과 `OrderServiceImpl`은 구현체에 의존하지 않고 오직 인터페이스에만 의존한다. 

![image-20230810145820670](/images/2023-08-10-Spring_principle_3/image-20230810145820670.png)

`MemberServiceImpl`과 `OrderServiceImpl`은 생성자를 통해 어떤 구현 객체가 주입될지 알 수 없고, 생성자를 통해 어떤 구현 객체를 주입할지는 오직 외부(`AppConfig`)에서 결정한다. 

`MemberServiceImpl`과 `OrderServiceImpl`은 이제부터 실행에만 집중하면 된다. 

<br>



**AppConfig 실행**

`MemberApp`

```java
public class MemberApp {
    public static void main(String[] args) {
      
        AppConfig appConfig = new AppConfig();
        // appConfig.memberService()는 MemoryMemberRepository() 객체를 생성함. 
        MemberService memberService = appConfig.memberService();

        Member member = new Member(1L, "memberA", Grade.DIAMOND);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);

        System.out.println("new member = " + member.getName());
        System.out.println("findMember = " + findMember.getName());
    }
}
```

<br>

`OrderApp`

```java
public class OrderApp {
    public static void main(String[] args) {

        AppConfig appConfig = new AppConfig();
       // appConfig.memberService()는 MemoryMemberRepository() 객체를 생성
        MemberService memberService = appConfig.memberService();
      
       // appConfig.orderService()는 MemoryMemberRepository()와 RateDiscountPolicy() 객체를 생성
        OrderService orderService = appConfig.orderService();

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.DIAMOND);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 20000);

        System.out.println("order = " + order);
    }
}
```



<br>

## 정리

- AppConfig를 통해서 애플리케이션이 크게 **사용 영역**과, 객체를 생성하고 **구성(Configuration)하는 영역**으로 분리하였다. 
- AppConfig는 구체 클래스를 선택한다. 애플리케이션이 어떻게 동작해야 할지 전체 구성을 책임진다.
- `OrderServiceImpl`은 기능을 실행하는 책임만 지면 된다. 
- 할인 정책을 변경하고 싶다면 AppConfig의 코드만 바꾸어주면 된다. 클라이언트 코드인 `OrderServiceImpl`를 포함해 사용 영역의 어떤 코드도 변경할 필요가 없다.



<br>



## 좋은 객체 지향 설계의 5가지 원칙 적용

여기서는 3가지 SRP, DIP, OCP를 적용하였다. 



<Br>

**SRP 단일 책임 원칙 - 한 클래스는 하나의 책임만 가져야 한다.**

- 구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당
- 클라이언트 객체는 실행하는 책임만 담당



**DIP 의존관계 역전 원칙 - 프로그래머는 "추상화에 의존해야지, 구체화에 의존하면 안된다"**

- 클라이언트 코드가 `DiscountPolicy` 추상화 인터페이스에만 의존하도록 코드를 변경했다.
- `AppConfig`가 `FixDiscountPolicy` 객체 인스턴스를 클라이언트 코드 대신 생성해서 클라이언트 코드에 의존관계를 주입했다. 이렇게해서 DIP 원칙을 따르면서 문제도 해결했다.



**OCP - 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다**

- 애플리케이션을 사용 영역과 구성 영역으로 나눔
- `AppConfig`가 의존관계를 `FixDiscountPolicy`->` RateDiscountPolicy` 로 변경해서 클라이언트 코드에 주입하므로 클라이언트 코드는 변경하지 않아도 됨
- **소프트웨어 요소를 새롭게 확장해도 사용 영역의 변경은 닫혀 있다!**



<br>



## IoC, DI, 그리고 컨테이너

**제어의 역전 IoC(Inversion of Control)**

- 프로그램에 대한 제어 흐름에 대한 권한은 모두 `AppConfig`가 가지고 있다. 심지어 `OrderServiceImpl` 도 `AppConfig`가 생성한다. 그리고 `AppConfig`는 `OrderServiceImpl` 이 아닌 `OrderService` 인터페이스의 다른 구현 객체를 생성하고 실행할 수 도 있다. 그런 사실도 모른체 `OrderServiceImpl` 은 묵묵히 자신의 로직을 실행할 뿐이다.
- 이렇듯 프로그램의 **제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것**을 **제어의 역전(IoC)**이라 한다.



<br>

**의존관계 주입 DI(Dependency Injection)**

- 애플리케이션 **실행 시점(런타임)**에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것을 **의존관계 주입**이라 한다.
- 의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.

<Br>

**IoC컨테이너, DI 컨테이너**

- AppConfig처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을 IoC컨테이너 또는 **DI 컨테이너**라고 한다.
- 의존관계 주입에 초점을 맞추어 최근에는 주로 DI컨테이너라 한다.

<Br>



## 스프링으로 전환하기

지금까지 순수한 자바 코드만으로 DI를 적용했는데, 이제 스프링을 사용해보자.

<br>

`AppConfig` - 스프링 기반으로 변경

```java
// import 생략

@Configuration // 애플리케이션의 설정 정보, 구성 정보를 뜻하는 애너테이션이다. 
public class AppConfig {

    @Bean // 스프링 컨테이너에 스프링 빈으로 등록한다. 메서드 이름(memberService)으로 등록된다. 
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean // 스프링 컨테이너에 스프링 빈으로 등록한다. 메서드 이름(memberRepository)으로 등록된다. 
    public static MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean // 스프링 컨테이너에 스프링 빈으로 등록한다. 메서드 이름(orderService)으로 등록된다. 
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean // 스프링 컨테이너에 스프링 빈으로 등록한다. 메서드 이름(discountPolicy)으로 등록된다. 
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy(); 
    }

}
```



<br>

`MemberApp`에 스프링 컨테이너 적용

```java
// import문 생략
public class MemberApp {
    public static void main(String[] args) {
//        AppConfig appConfig = new AppConfig();
//        MemberService memberService = appConfig.memberService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

        Member member = new Member(1L, "memberA", Grade.DIAMOND);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);

        System.out.println("new member = " + member.getName());
        System.out.println("findMember = " + findMember.getName());
    }
}
```



<br>

`OrderApp`에 스프링 컨테이너 적용

```java
// import문 생략
public class OrderApp {
    public static void main(String[] args) {

//        AppConfig appConfig = new AppConfig();
//        MemberService memberService = appConfig.memberService();
//        OrderService orderService = appConfig.orderService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);


        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.DIAMOND);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 20000);

        System.out.println("order = " + order);
    }
}
```

<br>

**스프링 컨테이너**

- 위 코드에 있는 `ApplicationContext`를 스프링 컨테이너라 한다. 기존에는 개발자가 `AppConfig`를 통해 직접 객체를 생성하고 DI를 했지만, 이제부터는 스프링 컨테이너를 통해서 사용한다.
- 스프링 컨테이너는 `@Configuration`이 붙은 `AppConfig`를 설정(구성)정보로 사용한다. 여기서 `@Bean`이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에 등록된 객체를 스프링 빈이라 한다.
- 스프링 빈은 `@Bean`이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다. 
- 이제부터는 필요한 객체를 스프링 컨테이너를 통해 필요한 스프링 빈(객체)를 찾아야 한다. 스프링 빈은 `applicationContext.getBean()` 메서드를 사용해서 찾을 수 있다.
- 기존에는 개발자가 직접 자바코드로 모든 것을 했다면, 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.