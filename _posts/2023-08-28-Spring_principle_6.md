---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(컴포넌트 스캔)"
categories: spring
author_profile: false
typora-root-url: ../







---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[인프런 김영한 님의 스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



-----------

이번 시간에는 컴포넌트 스캔에 대해서 알아볼 것이다.

## 컴포넌트 스캔과 의존관계 자동 주입 시작하기

- 지금까지 스프링 빈을 등록할 때 자바 코드의 `@Bean`을 통해서 설정 정보에 직접 등록할 스프링 빈을 나열했다.
- 예제에서는 직접 등록한 스프링 빈의 개수가 몇 개 되지 않았지만, 이 스프링 빈이 수십, 수백개가 되면 일일이 등록하기도 귀찮고, 누락되는 문제가 발생할 수 있다.
- 그래서 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
- 또한 의존관계도 자동으로 주입하는 `@AutoWired`라는 기능도 제공한다. 



<br>

`AutoAppConfig` 생성

```java
package hello.core;

@Configuration
@ComponentScan(    // 설정정보는 컴포넌트 스캔 대상에서 제외함.
        excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =
                Configuration.class))
public class AutoAppConfig {

}
```

- 컴포넌트 스캔을 사용하려면 먼저 `@ComponentScan`을 설정 정보에 붙여주면 된다.
- 기존의 `AppConfig`와 다르게 `@Bean`으로 등록한 클래스가 하나도 없다.

<br>

컴포넌트 스캔은 이름 그대로 `@Component` 애노테이션이 붙은 클래스를 스캔해서 스프링 빈으로 등록한다. 이제 각 클래스가 컴포넌트 스캔의 대상이 되도록 `@Component` 를 붙여주자.

<br>

`MemoryMemberRepository`  - @Component 추가

```java
@Component
public class MemoryMemberRepository implements MemberRepository {}
```



<br>

`RateDiscountPolicy` -  @Component 추가

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

<br>

`MemberServiceImpl` -  @Component, @Autowired 추가

```java
@Component
public class MemberServiceImpl implements MemberService{

    private final MemberRepository memberRepository;

    // 의존관계 자동 주입.
    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

- 이전에 `AppConfig`에서는 `@Bean`으로 직접 설정 정보를 작성하고 의존관계도 직접 명시했다.
- 이번에는 `AutoWired`를 통해 의존관계를 자동으로 주입하게 작성해주었다.

<br>

`OrderServiceImpl` - @Component, @AutoWired 추가

```java
@Component
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    // 의존관계 자동 주입. 생성자에서 여러 의존관계도 한번에 주입받을 수 있다.
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

<br>

`AutoAppConfigTest.java` - 테스트 작성

```java
public class AutoAppConfigTest {

    @Test
    void basicScan() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);

        MemberService memberService = ac.getBean(MemberService.class);
        Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
    }
}
```

- 실행해보면 기존과 같이 잘 동작하는 것을 확인할 수 있다.

<br>

컴포넌트 스캔과 자동 의존관계 주입이 어떻게 동작하는지 그림으로 알아보자.

1. @ComponentScan

   ![image-20230828162401351](/images/2023-08-28-Spring_principle_6/image-20230828162401351.png)

- `@ComponentScan`은 `@Component` 가 붙은 모든 클래스를 스프링 빈으로 등록한다.
- 스프링 빈의 기본 이름은 클래스명을 사용하되 맨 앞글자만 소문자를 사용한다. (MemberServiceImpl -> memberServiceImpl)
- 빈의 이름을 직접 지정하고 싶으면 `@Component("memberService2")` 의 형식으로 이름을 부여하면 된다.

<br>

2. `@AutoWired` 의존관계 자동 주입

   ![image-20230828162712755](/images/2023-08-28-Spring_principle_6/image-20230828162712755.png)

- 생성자에 `@AutoWired`를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
- 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다. `getBean(MemberRepository.class)`와 동일하다고 이해하면 된다.
- 생성자에 파라미터가 많아도 다 찾아서 자동으로 주입한다.



<br>

## 탐색 위치와 기본 스캔 대상

모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다. 그래서 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.

```java
@ComponentScan (
					basePackages = "hello.core")
```

- `basePackages`는 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색한다.
- `basePackages = {"hello.core", "hello.service"}`처럼 여러 시작 위치를 정할 수 있다.
- 만약 아무것도 지정하지 않으면 `@ComponentScan`이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.

<br>

### 컴포넌트 스캔 기본 대상

컴포넌트 스캔은 `@Component`뿐만 아니라 다음 내용도 추가로 대상에 포함된다.

- `@Component` : 컴포넌트 스캔에서 사용
- `@Controller` : 스프링 MVC 컨트롤러에서 사용
- `@Service` : 스프링 비즈니스 로직에서 사용
- `@Repository` : 스프링 데이터 접근 계층에서 사용
- `@Configuration` : 스프링 설정 정보에서 사용

<br>

## 필터

- `includeFilters` : 컴포넌트 스캔 대상을 추가로 지정한다.
- `excludeFilters` : 컴포넌트 스캔에서 제외할 대상을 지정한다.

<br>

### FilterType 옵션

- ANNOTATION : 기본값, 애노테이션을 인식해서 동작한다.
- ASSIGNABLE_TYPE : 지정한 타입과 자식 타입을 인식해서 동작한다.
- ASPECTJ : ASpectJ 패턴 사용
- REGEX : 정규 표현식
- CUSTOM : `TypeFilter`라는 인터페이스를 구현해서 처리 



<br>

### 수동 빈 등록 vs 자동 빈 등록

수동 빈 등록과 자동 빈 등록에서 빈 이름이 충돌된다면 수동 빈 등록이 우선권을 가진다.(수동 빈이 자동 빈을 오버라이딩 한다.)

하지만 개발자가 의도해서 수동 빈 등록이 오버라이딩 되는 결과가 만들어지기 보다는 여러 설정들이 꼬이는 경우가 많이 발생한다. 그래서 최근 스프링 부트에서는 수동 빈 등록과 자동 빈 등록이 충돌나면 오류가 발생하도록 기본 값을 바꾸었다. 















