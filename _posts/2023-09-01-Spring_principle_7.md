---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(의존관계 자동 주입)"
categories: spring
author_profile: false
typora-root-url: ../








---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[인프런 김영한 님의 스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



-----------

이번시간에는 **의존관계 자동 주입**에 대해 알아보자.



<br>

## 다양한 의존관계 주입 방법

의존관계 주입에는 크게 4가지 방법이 있다.

- 생성자 주입
- 수정자 주입(setter 주입)
- 필드 주입
- 일반 메서드 주입



<br>

**생성자 주입**

- 생성자를 통해 의존 관계를 주입 받는 방법이다.
- 지금까지 우리가 진행했던 방법이 바로 생성자 주입이다.
- 특징으로는 생성자 호출시점에 딱 1번만 호출되는 것이 보장되고, **불변, 필수** 의존관계에 사용된다.

```java
@Component
public class OrderServiceImpl implements OrderService {
		private final MemberRepository memberRepository;
 		private final DiscountPolicy discountPolicy;
 
  // 생성자가 하나만 있으므로 @Autowired 생략 가능.
  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
 			this.memberRepository = memberRepository;
 			this.discountPolicy = discountPolicy;
	 }
}
```



<br>

**수정자 주입(setter 주입)**

- setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해 의존관계를 주입하는 방법이다. 
- **선택, 변경** 가능성이 있는 의존관계에 사용되고, 자바빈 프로퍼티 규악의 수정자 메서드 방식을 사용하는 방법이다. 

```java
@Component
public class OrderServiceImpl implements OrderService {
  
		private MemberRepository memberRepository;
		private DiscountPolicy discountPolicy;
  
		@Autowired
		public void setMemberRepository(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
		}
  
		@Autowired
		public void setDiscountPolicy(DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy;
		}
}
```

- `@Autowired`의 기본 동작은 주입할 대상이 없으면 오류가 발생한다. 주입할 대상이 없어도 동작하게 하려면 `@Autowired(required = false)`로 지정하면 된다. 



<br>

**필드 주입**

- 이름 그대로 필드 그대로 주입하는 방법이다.
- 코드가 간결하지만 외부에서 변경이 불가능해서 테스트 하기 힘들다는 치명적 단점이 있다.
- 사용하지 말자. 



<br>

**일단 메서드 주입**

- 일반 메서드를 통해 주입 받을 수 있다.
- 한 번에 여러 필드를 주입 받을 수 있다. 일반적으로 잘 사용하지 않는다. 

```java
@Component
public class OrderServiceImpl implements OrderService {
  
		private MemberRepository memberRepository;
		private DiscountPolicy discountPolicy;
  
		@Autowired
		public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
		this.memberRepository = memberRepository;
		this.discountPolicy = discountPolicy;
		}
}
```

<br>

###  생성자 주입을 선택하자.

최근에는 스프링을 포함한 DI 프레임워크 대부분이 생성자 주입을 권장한다. 그 이유는 다음과 같다.

**불변**

- 대부분의 의존관계는 주입이 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없다. 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.
- 수정자 주입을 사용하면 setXxx 메서드를 public으로 열어두어야 하는데, 누군가 실수로 변경할 가능성도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.
- 생성자 주입은 객체를 생성할 때 딱 1번만 호출되므로 이후에 호출되는 일이 없어서 불변하게 설계할 수 있다.

<br>

**정리**

- 생성자 주입을 사용하면 필드에 `final`키워드를 사용해 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아줄 수 있다.
- 생성자 주입 방식은 프레임워크에 의존하지 않고, 순수한 자바 언어의 특징을 잘 살리는 방법이다.
- 기본적으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에는 수정자 주입 방식을 옵션으로 부여하자.

<br>

## 옵션 처리

주입할 스프링 빈이 없어도 동작해야 할 때가 있는데, 자동 주입 대상을 옵션으로 처리하는 방법이 있따.

- `@Autowired(required=false)` : 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안됨
- `org.springframework.lang.@Nullable` : 자동 주입할 대사잉 없으면 null이 입력된다.
- `Optional<>` : 자동 주입할 대상이 없으면 `Optional.empty`가 입력된다. 

**예제**

```java
//호출 안됨
@Autowired(required = false)
public void setNoBean1(Member member) {
		System.out.println("setNoBean1 = " + member);
}

//null 호출
@Autowired
public void setNoBean2(@Nullable Member member) {
		System.out.println("setNoBean2 = " + member);
}

//Optional.empty 호출
@Autowired(required = false)
public void setNoBean3(Optional<Member> member) {
		System.out.println("setNoBean3 = " + member);
}
```

**실행 결과**

```java
setNoBean2 = null
setNoBean3 = Optional.empty
```



<br>

## 롬복과 최신 트렌드

개발을 하다 보면, 대부분이 다 불변이기 때문에 필드에 `final`키워드를 사용하게 된다. 그런데 생성자고 만들어야 하고, 주입 받은 값을 대입하는 코드도 만들어야 한다. 이것을 `롬복`을 이용해 최적화할 수 있다.

**기본 코드**

```java
@Component
public class OrderServiceImpl implements OrderService {
  
		private final MemberRepository memberRepository;
		private final DiscountPolicy discountPolicy;
  
		@Autowired
		public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
		this.memberRepository = memberRepository;
		this.discountPolicy = discountPolicy;
		}
}
```



<br>

여기에 롬복을 적용하여 롬복 라이브러리가 제공하는 `@RequiredArgsConstructor`기능을 사용하면 final이 붙은 필드를 모아 생성자를 자동으로 만들어준다. 

**최종 결과 코드**

```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {
		private final MemberRepository memberRepository;
		private final DiscountPolicy discountPolicy;		
}
```

이 최종결과 코드와 이전의 코드는 완전히 동일하다. 롬복이 자바의 애노테이션 프로세서라는 기능을 이용하여 컴파일 시점에 생성자 코드를 자동으로 생성해준다.

<br>

최근에는 생성자를 딱 1개 두고, `@Autowired` 를 생략하는 방법을 주로 사용한다. 여기에 Lombok 라이브러리의 `@RequiredArgsConstructor` 함께 사용하면 기능은 다 제공하면서, 코드는 깔끔하게 사용할 수 있다



<br>

이번에는 조회한 빈이 2개 이상일 때 오류가 발생하면, 해결하는 3가지 방법에 대해서 알아보자.

## @Autowired 필드 명,  @Qualifier, @Primary

조회 대상 빈이 2개 이상일 때 해결 방법

- @Autowired 필드 명 매칭
- @Qualifier -> @Qualifier끼리 매칭 -> 빈 이름 매칭
- @Primary 사용



<br>

**@Autowired 필드 명 매칭**

`@Autowired`는 타입 매칭을 시도하고, 이때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.

```java
//기존 코드
@Autowired
private DiscountPolicy discountPolicy

// 필드 명을 빈 이름으로 변경
@Autowired
private DiscountPolicy rateDiscountPolicy
```

필드 명이 `rateDiscountPolicy`이므로 정상 주입된다.

**필드 명 매칭은 먼저 타입 매칭을 시도 하고 그 결과에 여러 빈이 있을 때 추가로 동작하는 기능이다.**

<br>

**@Qualifer 사용**

`@Qualifier`는 추가 구분자를 붙여주는 방법이다. 주입시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것은 아니다.

<br>

빈 등록시 @Qualifier를 붙여 준다.

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {}
```

주입 시에 @Qualifier를 붙여주고 등록한 이름을 적어준다. 

<br>

**생성자 자동 주입 예시**

```java
@Autowired
public OrderServiceImpl(MemberRepository memberRepository,@Qualifier("mainDiscountPolicy") DiscountPolicy
discountPolicy) {
		this.memberRepository = memberRepository;
		this.discountPolicy = discountPolicy;
}
```



<br>

`@Qualifier`는 `@Qualifier`를 찾는 용도로만 사용하는게 병확하고 좋다.

<br>

**@Primary 사용**

`@Primary`는 우선순위를 정하는 방법이다. `@Autowired`시에 여러 빈이 매칭되면 `@Primary`가 우선권을 가진다. 

```java
Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

<br>

**우선순위**

`@Primary`는 기본값 처럼 동작하는 것이도, `@Qualifier`는 매우 상세하게 동작한다. 스프링은 자동보다는 수동이, 넓은 범위의 선택권 보다는 좁은 범위의 선택권이 우선 순위가 높다. 따라서 `@Qualifier`가 우선권이 높다.



<br>

## 자동, 수동의 올바른 실무 운영 기준

편리한 자동 기능을 기본으로 사용하고, 직접 등록하는 기술 지원 객체는 수동 등록하자.

다형성을 적극 활용하는 비즈니스 로직은 수동 등록을 고민해보자.















