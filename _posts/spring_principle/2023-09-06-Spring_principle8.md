---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(빈 생명주기 콜백)"
categories: spring
author_profile: false
typora-root-url: ../



---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[인프런 김영한 님의 스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



-----------

이번 시간에는 빈 생명주기 콜백에 대해서 알아보자.

## 빈 생명주기 콜백 시작

데이터베이스 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, 객체의 초기화와 종료 작업이 필요하다.

<br>

간단하게 외부 네트워크에 미리 연결하는 객체를 하나 생성한다고 가정해보자. 실제 네트워크에 연결되는 것은 아니며, 단순 문자만 출력되도록 한다. 이 `NetworkClient`는 애플리케이션 시작 시점에 `connect()`를 호출하여 연결을 맺어두어야 하고, 애플리케이션이 종료되면 `disconnect()`를 호출하여 연결을 끊어야 한다.

```java
package hello.core.lifecycle;

public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println(" 생성자 호출, url = " + url);
      	connect();
    		call("초기화 연결 메시지")
    }

    public void setUrl(String url) {
        this.url = url;
    }

    //서비스 시작시 호출
    public void connect() {
        System.out.println("connect : " + url);
    }

    public void call(String message) {
        System.out.println("call: " + url + " message = " + message);
    }

    //서비스 종료시 호출
    public void disconnect() {
        System.out.println("close: " + url);
    }
```

<br>

**환경설정과 실행**

```java
package hello.core.lifecycle;

public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifeCycleConfig {

        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://minki-dev");
            return networkClient;
        }
    }
}

```

실행하면 이상한 결과가 나오게 된다.

```
생성자 호출, url = null
connect: null
call: null message = 초기화 연결 메시지
```

생성자 부분에서 url정보 없이 connect가 호출되기 때문이다. 객체를 생성하는 단계에는 url이 없고, 객체를 생성한 다음 외부에서 수정자 주입을 통해 `setUrl()`이 호출되어야 url가 존재하게 된다.

<br>

스프링 빈은 `객체 생성 -> 의존관계 주입`의 라이프사이클을 가진다.

<br>

그런데 개발자는 의존관계 주입이 모두 완료된 시점을 알 수가 없지만 **스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공**한다. 또한 **스프링은 스프링 컨테이너가 종료되기 직전에 소멸을 콜백**해준다. 따라서 안전하게 종료 작업을 진행할 수 있다. 

<br>

**스프링 빈의 이벤트 라이프사이클**

스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸전 콜백 -> 스프링 종료

<br>

**스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원한다**

- 인터페이스(InitializingBean, DisposalBean)
- 설정 정보에 초기화 메서드, 종료 메서드 지정
- @PostConstruct, @PreDestroy 애노테이션 지원

위 3가지 방법 중에서 가장 많이 사용하는 애노테이션을 이용한 @PostConstruct, @PreDestroy에 대해서 알아보자.

<br>

## 애노테이션 @PostConstruct, @PreDestroy

```java
package hello.core.lifecycle;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println(" 생성자 호출, url = " + url);
    }

    public void setUrl(String url) {
        this.url = url;
    }

    //서비스 시작시 호출
    public void connect() {
        System.out.println("connect : " + url);
    }

    public void call(String message) {
        System.out.println("call: " + url + " message = " + message);
    }

    //서비스 종료시 호출
    public void disconnect() {
        System.out.println("close: " + url);
    }

    @PostConstruct
    public void init() {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 연결 메시지");
    }

    @PreDestroy
    public void close() {
        System.out.println("NetworkClient.close");
        disconnect();
    }
}

```

```java
@Configuration
static class LifeCycleConfig {

	@Bean
	public NetworkClient networkClient() {
	NetworkClient networkClient = new NetworkClient();
	networkClient.setUrl("http://minki-dev");
	return networkClient;
	}
}
```

**실행 결과**

```
 생성자 호출, url = null
NetworkClient.init
connect : http://minki-dev
call: http://minki-dev message = 초기화 연결 메시지
14:37:45.671 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@4659191b, started on Wed Sep 06 14:37:45 KST 2023
NetworkClient.close
close: http://minki-dev
```

`@PostConstruct`, `@PreDestroy` 이 두 애노테이션을 사용하면 가장 편리하게 초기화와 종료를 실행할 수 있다.

<br>
**@PostConstruct, @PreDestroy** 애노테이션 특징

- 최신 스프링에서 가장 권장하는 방법
- 애노테이션 하나만 붙이면 되므로 매우 편리함
- 패키지를 보면 `javax.annotation.PostConstruct`이다. 스프링에 종속적인 기술이 아닌 JSR-250라는 자바 표준임. 따라서 스프링이 아닌 다른 컨테이너에서도 동작함
- 컴포넌트 스캔과 잘 어울림
- 유일한 단점은 외부 라이브러리에 적용하지 못하는다는 것. 외부 라이브러리를 초기화, 종료 해야 하면 `@Bean` `initMethod`, `destroyMethod` 의 기능을 사용하자







