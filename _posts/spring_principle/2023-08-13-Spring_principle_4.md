---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(스프링 컨테이너와 스프링 빈)"
categories: spring
author_profile: false
typora-root-url: ../





---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[인프런 김영한 님의 스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



-----------

이번 시간에는 스프링 컨테이너와 스프링 빈에 대해서 알아볼 것이다.



<br>

`스프링 컨테이너 생성`

```java
// 스프링 컨테이너 생성
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);

```

- 위에 있는 `ApplicationContext`를 스프링 컨테이너라 하며, 인터페이스이다.
- 스프링 컨테이너는 XML 기반으로 만들 수도 있고, 애노테이션 기반의 자바 설정 클래스도 만들 수 있다.
- `AppConfig`를 사용했던 방식이 애노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것이다.
- `new AnnotationConfigApplicationContext(AppConfig.class);` 이 클래스는 `ApplicationContext`인터페이스의 구현체이다. 



<br>

### 스프링 컨테이너의 생성 과정

1. 스프링 컨테이너 생성![image-20230813170803542](/images/2023-08-13-Spring_principle_4/image-20230813170803542.png)

- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 하는데, 여기서는 `AppConfig.class`를 구성 정보로 지정했다.

<br>

2. 스프링 빈 등록![image-20230813170918121](/images/2023-08-13-Spring_principle_4/image-20230813170918121.png)

- 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다. 
- 빈 이름은 메서드 이름을 사용하는데, `@Bean(name="memberService2")처럼 직접 부여할 수도 있다.
- 빈 이름은 항상 다른 이름을 부여해야 한다. 

<br>

3. 스프링 빈 의존관계 설정 - 준비![image-20230813171131179](/images/2023-08-13-Spring_principle_4/image-20230813171131179.png)



<br>

4. 스프링 빈 의존관계 설정 - 완료 ![image-20230813171151669](/images/2023-08-13-Spring_principle_4/image-20230813171151669.png)

- 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입(DI)한다.



### 스프링 컨테이에 등록된 애플리케이션 빈 출력하기

`hello.core.beanfind.ApplicationContextInfoTest`

```java
public class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("애플리케이션 출력하기")
    void findApplicationBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            //Role ROLE_APPLICATION: 직접 등록한 애플리케이션 빈
            //Role ROLE_INFRASTRUCTURE: 스프링이 내부에서 사용하는 빈
            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + " object = " + bean);
            }
        }
    }
}
```

- 스프링이 내부에서 사용하는 빈은 제외하고, 내가 등록한 빈만 `getRole()`로 구분하여 출력해보았다.

<br>

### 스프링 빈 조회 - 상속 관계

![image-20230813171640351](/images/2023-08-13-Spring_principle_4/image-20230813171640351.png)

- 부모 타입으로 조회하면, 자식 타입도 함께 조회된다.
- 그래서 모든 자바 객체의 최고 부모인 `Object` 조회하면, 모든 스프링 빈이 조회된다. 

<br>



### BeanFactory와 ApplicationContext

<img src="/images/2023-08-13-Spring_principle_4/image-20230813171826513.png" alt="image-20230813171826513" style="zoom:50%;" />

**BeanFactory**

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리하고 조회하는 역할을 담당
- `getBean()`을 제공

<br>

**ApplicationContext**

- BeanFactory 기능을 모두 상속받아서 제공

- <img src="/images/2023-08-13-Spring_principle_4/image-20230813171953561.png" alt="image-20230813171953561" style="zoom:50%;" />

- **메시지소스를 활용한 국제화 기능** - 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
- **환경변수** - 로컬, 개발, 운영등을 구분해서 처리
- **애플리케이션 이벤트** - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- **편리한 리소스 조회** - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

<br>

**정리**

- ApplicationContext는 BeanFactory의 기능을 상속받아 빈 관리기능 + 편리한 부가 기능을 제공함
- BeanFactory나 ApplicationContext를 스프링 컨테이너라 한다.



<br>



### 다양한 설정 형식을 지원하는 스프링 컨테이너

- <img src="/images/2023-08-13-Spring_principle_4/image-20230813172232567.png" alt="image-20230813172232567" style="zoom:50%;" />
- 스프링 컨테이너는 다양한 형식(자바 코드, XML, Groovy)의 설정 정보를 받아드릴 수 있게 유연하게 설계되어 있다.



<br>

### 스프링 빈 설정 메타 정보 - BeanDefinition

- 스프링은 `BeanDefinition`이라는 추상화가 있기 때문에 다양한 설정 형식을 지원한다.
- 쉽게 말해 **역할과 구현을 개념적으로 나눈 것**
- `BeanDefinition`을 빈 설정 메타정보라 한다. 
- 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다. 
- <img src="/images/2023-08-13-Spring_principle_4/image-20230813172542084.png" alt="image-20230813172542084" style="zoom:50%;" />
- BeanDefinition에 대해 너무 깊이있게 이해하기 보다는, 스프링이 다양한 형태의 설정 정보를 BeanDefinition으로 추상화해서 사용하는 것 정도만 이해하자.























