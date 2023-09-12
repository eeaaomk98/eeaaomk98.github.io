---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(예제 만들기)"
categories: spring
author_profile: false
typora-root-url: ../




---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



이번시간에는 역할과 구현을 나누어 예제를 만들어보자.



## 프로젝트 생성

<img src="/images/2023-08-07-Spring_principle_2/capture 23.png" alt="capture 23"  />

- Project : Gradle-Groovy 
- Language : Java
- Spring Boot : 2.8.14
- Packaging : Jar
- Java : 11

https://start.spring.io 사이트에서 설정을 해주고 Generate를 눌러 압축파일을 다운로드해주었다.

<br>

파일을 압축해제하고, 인텔리제이에서 build.gradle로 open 해주었다. 

![image-20230807152446566](/images/2023-08-07-Spring_principle_2/image-20230807152446566.png)



## 비즈니스 요구사항과 설계


**회원**

- 회원을 가입하고 조회할 수 있다.
- 회원은 BRONZE와 DIAMOND 두 가지 등급이 있다.
- 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)

**주문과 할인 정책**

- 회원은 상품을 주문할 수 있다.
- 회원 등급에 따라 할인 정책을 적용할 수 있다.
- 할인 정책은 모든 VIP는 2000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)
- 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)

<br>

이 주어진 요구사항을 기반으로 인터페이스를 만들고 구현체를 언제든지 갈아끼울 수 있도록 설계해보자.

<br>

## 회원 도메인 설계

<img src="/images/2023-08-07-Spring_principle_2/image-20230807210645719.png" alt="image-20230807210645719" style="zoom: 67%;" />

역할과 구현을 나누어 인터페이스와 구현클래스로 나누었고, `MemberRepository`의 구현체로는 `MemoryMemberRepository`로 설정하였다. 



<br>

## 회원 도메인 개발

<br>

`hello.core.member.Grade` - 회원 등급

```java
package hello.core.member;

// BRONZE, DIAMOND 두 등급이 존재.
public enum Grade {
    BRONZE,
    DIAMOND
}
```



<br>



`hello.core.member.Member` - 회원 엔티티

```java
package hello.core.member;

public class Member {

    private Long id;
    private String name;
    private Grade grade;

    public Member(Long id, String name, Grade grade) {
        this.id = id;
        this.name = name;
        this.grade = grade;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Grade getGrade() {
        return grade;
    }

    public void setGrade(Grade grade) {
        this.grade = grade;
    }
}
```

<br>



**회원 저장소**

`hello.core.member.MemberRepository` - 회원 저장소 인터페이스(역할)

```java
package hello.core.member;

public interface MemberRepository {

    void save(Member member);       // 저장

    Member findById(Long memberId); // 조회
}
```

<br>

`hello.core.member.MemberServiceImpl` - 메모리 회원 저장소 구현체

```java
package hello.core.member;

import java.util.HashMap;
import java.util.Map;

public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<>();


    @Override
  // member.id를 key로, member를 value로 하여 store에 저장(Map 형식)
    public void save(Member member) {
        store.put(member.getId(), member);
    }

    @Override
  // memberId를 통해 member 객체 찾기
    public Member findById(Long memberId) {
        return store.get(memberId);
    }
}
```

<br>



**회원 서비스**

`hello.core.member.MemberService` - 회원 서비스 인터페이스

```java
package hello.core.member;

public interface MemberService {

    void join(Member member);

    Member findMember(Long memberId);
}
```

<br>

`hello.core.member.MemberServiceImpl` - 회원 서비스 구현체

```java
package hello.core.member;

public class MemberServiceImpl implements MemberService{

  // 회원을 가입하고 조회하기 위해 MemoryMemberRepository가 필요. 
    private final MemberRepository memberRepository = new MemoryMemberRepository();

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

<br>

이렇게 작성한 비즈니스 로직을 통해 회원 도메인 실행과 테스트를 해보자. 

## 회원 도메인 실행과 테스트

`hello.core.MemberApp` - 회원 가입 

```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;

public class MemberApp {
    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member(1L, "memberA", Grade.DIAMOND);
        memberService.join(member);  // id=1L, name="memberA", grade="DIAMOND"로 회원 가입.

        // id=1L로 member를 찾아 findMember에 저장
        Member findMember = memberService.findMember(1L); 

        System.out.println("new member = " + member.getName());
        System.out.println("findMember = " + findMember.getName());
    }
}
```



결과값으로 `member.getName()`과 `findMember.getName()`의 값이 일치하는 결과가 나왔다. 

원하는 결과가 나온 것은 맞지만, 애플리케이션 로직으로 이렇게 테스트 하는 것은 바람직한 방법은 아니다. 

JUnit을 통해 테스트 코드를 작성해보자.  

![image-20230807212520376](/images/2023-08-07-Spring_principle_2/image-20230807212520376.png)

<br>

`src.main.`이 아닌 `src.test`에서 파일을 만든다는 것을 잘 알고 있어야 한다. 

![image-20230807213000471](/images/2023-08-07-Spring_principle_2/image-20230807213000471.png)

`hello.core.member.MemberServiceTest` - JUnit을 활용한 회원 가입 테스트 

```java
package hello.core.member;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class MemberServiceTest {

    MemberService memberService = new MemberServiceImpl();

    @Test
    void join() {
        Member member = new Member(1L, "memberA", Grade.DIAMOND);

        memberService.join(member);
        Member findMember = memberService.findMember(1L);

      // Assertions를 이용한 검증(member와 findMember가 같은지)
        Assertions.assertThat(member).isEqualTo(findMember);
    }
}
```

![image-20230807191826299](/images/2023-08-07-Spring_principle_2/image-20230807191826299.png)

`System.out.println()`이 아닌 애노테이션 `@Test`를 통해서 테스트 코드를 작성하도록 한다.

검증은 `assertj.core.api.Assertions`를 이용하여 `member`와 `findMember`가 같은지 확인하였다. 



<br>

**회원 도메인 설계의 문제점**

![capture 29](/images/2023-08-07-Spring_principle_2/capture 29.png)

현재 의존관계가 인터페이스 뿐만 아니라 구현까지 모두 의존하는 문제점이 있다. 이것은 차근차근 해결해보도록 하자.



<br>

## 주문과 할인 도메인 설계

`주문 도메인 협력, 역할, 책임`

<img src="/images/2023-08-07-Spring_principle_2/image-20230807214300164.png" alt="image-20230807214300164" style="zoom:80%;" />

**1. 주문 생성** : 클라이언트는 주문 서비스에 주문 생성을 요청한다.

**2. 주문 조회** : 할인을 위해서는 회원 등급이 필요하다. 그래서 주문 서비스는 회원 저장소에서 회원을 조회한다.

**3. 할인 적용** : 주문 서비스는 회원 등급에 따른 할인 여부를 할인 정책에 위임한다.

**4. 주문 결과 확인** : 주문 서비스는 할인 결과를 포함한 주문 결과를 반환한다.

참고 : 실제로는 주문 데이터를 DB에 저장하겠지만, 이 예제에서는 단순히 주문결과를 반환한다. 



<br>



`주문 도메인 전체`

<img src="/images/2023-08-07-Spring_principle_2/image-20230807214522086.png" alt="image-20230807214522086" style="zoom:80%;" />

`역할과 구현을 분리`하여 자유롭게 구현 객체를 조립할 수 있게 설계했다. 



<br>

`주문 도메인 클래스 다이어그램`

<img src="/images/2023-08-07-Spring_principle_2/image-20230807214634200.png" alt="image-20230807214634200" style="zoom:80%;" />







<br>

`주문 도메인 객체 다이어그램1`

<img src="/images/2023-08-07-Spring_principle_2/image-20230807214709206.png" alt="image-20230807214709206" style="zoom:80%;" />

해당 다이어그램 형식에 맞게 코드를 작성해보자.

<br>

### 주문과 할인 도메인 개발

`hello.core.discount.DiscountPolicy` - 할인 정책 인터페이스(역할)

```java
package hello.core.discount;

import hello.core.member.Member;

public interface DiscountPolicy {

    /**
     * @return 할인 대상 금액
     */
    int discount(Member member, int price);
}
```



<br>

`hello.core.discount.FixDiscountPolicy` - 정액 할인 정책 구현체

```java
package hello.core.discount;

import hello.core.member.Grade;
import hello.core.member.Member;

public class FixDiscountPolicy implements DiscountPolicy{

    private int discountFixAmount = 2000; // 2000원 할인


    // Grade 가 DIAMOND 이면 2000원 할인(반환), 아니면 할인 없음 
    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.DIAMOND) {
            return discountFixAmount;
        } else {
            return 0;
        }
    }
}
```



<br>

`hello.core.order.Order` - 주문 엔티티

```java
package hello.core.order;

public class Order {

    private Long memberId;
    private String itemName;
    private int itemPrice;
    private int discountPrice;

    public Order(Long memberId, String itemName, int itemPrice, int discountPrice) {
        this.memberId = memberId;
        this.itemName = itemName;
        this.itemPrice = itemPrice;
        this.discountPrice = discountPrice;
    }

    // 최종 계산금액 반환
    public int calculatePrice() {
        return itemPrice - discountPrice;
    }

    public Long getMemberId() {
        return memberId;
    }

    public void setMemberId(Long memberId) {
        this.memberId = memberId;
    }

    public String getItemName() {
        return itemName;
    }

    public void setItemName(String itemName) {
        this.itemName = itemName;
    }

    public int getItemPrice() {
        return itemPrice;
    }

    public void setItemPrice(int itemPrice) {
        this.itemPrice = itemPrice;
    }

    public int getDiscountPrice() {
        return discountPrice;
    }

    public void setDiscountPrice(int discountPrice) {
        this.discountPrice = discountPrice;
    }

   // toString()을 오버라이딩하여 Order출력 시 직관적으로 보이도록 함.
    @Override
    public String toString() {
        return "Order{" +
                "memberId=" + memberId +
                ", itemName='" + itemName + '\'' +
                ", itemPrice=" + itemPrice +
                ", discountPrice=" + discountPrice +
                '}';
    }
}
```

<br>



`hello.core.order.OrderService` - 주문 서비스 인터페이스(역할)

```java
package hello.core.order;

public interface OrderService {
    Order createOrder(Long memberId, String itemName, int itemPrice);
}
```



<br>

`hello.core.order.OrderServiceImpl` - 주문 서비스 구현체

```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;

public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    @Override
    // memberId로 객체를 찾아 member에 저장하고, 할인 정책을 적용한 다음 주문 객체를 생성하여 반환
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```





<br>

### JUnit을 활용한 주문과 할인 정책 테스트

`hello.core.order.OrderServiceTest` 

```java
package hello.core.order;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class OrderServiceTest {

    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();

    @Test
    void createOrder() {
        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.DIAMOND);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);
        Assertions.assertThat(order.getDiscountPrice()).isEqualTo(2000);
    }
}
```

`createOrder(memberId, "itemA", 10000)`을 통해 `new Order(1L, "itemA", 10000, 20000)`가 `order`에 저장된다. 

때문에 `order.getDiscountPrice()`는 2000원과 같은 값이므로 검증에 성공하게 된다.



<br>

이번 시간에는 비즈니스 요구사항에 맞게 로직을 작성하였고, 주문과 할인 설계 부분도 작성하여 테스트까지 마쳐 보았다. 

다음 시간에는 이번 시간에 해결하지 못한 의존관계 문제점에 대하여 해결해보자.











