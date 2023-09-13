---
published: true
layout: single
title: "스프링 핵심원리 - 기본편(빈 스코프)"
categories: spring
author_profile: false
typora-root-url: ../




---

해당 게시물은 인프런 김영한님의 스프링 핵심 원리-기본편을 참고하였습니다.

[인프런 김영한 님의 스프링 핵심원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)



-----------

이번시간에는 **빈 스코프**에 대해 알아보자.

<br>

## 빈 스코프란 ?

스코프는 번역 그대로 빈이 존재할 수 있는 범위를 뜻한다.

<br>

**스프링은 다음과 같은 다양한 스코프를 지원한다.**

- **싱글톤** : 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다.
- **프로토타임** : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프이다.
- **웹 관련 스코프**
    - **request** : 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다.
    - **session** : 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프이다.
    - **application** : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프이다.



<br>

빈 스코프는 다음과 같이 지정할 수 있다.

**컴포넌트 스캔 자동 등록**

```java
// 스코프를 "prototype"으로 지정
@Scope("prototype")
@Component
public class HelloBean {}
```

<br>

**수동 등록**

```java
@Scope("prototype")
@Bean
PrototypeBean HelloBean() {
	return new HelloBean();
}
```



<br>

## 프로토타입 스코프

x프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.

**프로토타입 빈 요청1**

![image-20230908164217927](/images/2023-09-08-Spring_principle9/image-20230908164217927.png)

- 1 . 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다
- 2 . 스프링 컨테이너에는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입한다.

<br>

**프로토타입 빈 요청 2**

![image-20230908164347209](/images/2023-09-08-Spring_principle9/image-20230908164347209.png)

- 3 . 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
- 4 . 이후에 스프링 컨테이너는 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환한다.

<br>

핵심은 **스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다는 것**이다. 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있다. 그래서 `@PreDestroy`같은 종료 메서드가 호출되지 않는다. 

<br>

**프로토타입 빈의 특징 정리**

- 스프링 컨테이너에 요청할 떄 마다 새로 생성된다.
- 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입 그리고 초기화까지만 관여한다.
- 종료 메서드가 호출되지 않는다.
- 그래서 프로토타입 빈은 프로토타입 빈을 조회한 클라이언트가 관리해야 한다. 종료 메서드에 대한 호출도 클라이언트가 직접 해야한다.

<br>

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

`clientBean`이라는 싱글톤 빈이 의존관계 주입을 통해서 프로토타입 빈을 주입받아서 사용하는 예를 보자.

**싱글톤에서 프로토타입 빈 사용1**

![image-20230908191334465](/images/2023-09-08-Spring_principle9/image-20230908191334465.png)

- `clientBean`은 싱글톤이므로, 보통 스프링 컨테이너 생성 시점에 함께 생성되고, 의존관계 주입도 발생한다.
- 1 . `clientBean`은 의존관계 자동 주입을 사용한다. 주입 시점에 스프링 컨테이너에 프로토타입 빈을 요청한다.
- 2 . 스프링 컨테이너는 프로토타입 빈을 생성해서 `clientBean`에 반환한다. 프로토타입 빈의 count 필드값은 0이다.
- 이제 `clientBean`은 프로토타입 빈을 내부 필드에 보관한다. (정확히는 참조값을 보관)

<br>

**싱글톤에서 프로토타입 빈 사용2**

![image-20230908191551742](/images/2023-09-08-Spring_principle9/image-20230908191551742.png)

- 클라이언트 A는 `clientBean`을 스프링 컨테이너에 요청해서 받는다. 싱글톤이므로 항상 같은 `clientBean`이 반환된다.
- 3 . 클라이언트 A는 `clientBean.logic()`을 호출한다.
- 4 . `clientBean`은 prototypeBean의 `addCount()`를 호출해서 프로토타입 빈의 count를 증가하여 count값이 1이 된다.

<br>

**싱글톤에서 프로토타입 빈 사용3**

![image-20230908191900003](/images/2023-09-08-Spring_principle9/image-20230908191900003.png)

- 클라이언트 B는 `clientBean`을 스프링 컨테이너에 요청해서 받는다. 싱글톤이므로 항상 같은 `clientBean`이 반환된다.
- **중요한 점은 clientBean이 내부에 가지고 있는 프로토타입 빈은 이미 과거에 주입이 끝난 빈이다. 주입 시점에 스프링 컨테이너에 요청해서 프로토타입 빈이 새로 생성이 된 것인지, 사용 할 때마다 새로 생성되는 것이 아니다!**
- 5 . 클라이언트 B는 `clientBean.logic()`을 호출한다.
- 6 . `clientBean`은 prototypeBean의 `addCount()`를 호출해서 프로토타입 빈의 count를 증가한다. 원래 count값이 1이었으므로 2가 된다. 



싱글톤 빈은 생성 시점에만 의존관계 주입을 받기 때문에, 프로토타입이 빈이 새로 생성되기는 하지만, 싱글톤 빈과 함께 계속 유지되는 것이 문제다. 

이 문제를 Provider를 통해 해결할 수 있다.

## ObjectFactory, ObjectProvider

지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 바로 `ObjectProvier`이다. 과거에는 `ObjectFactory`가 있었는데, 여기에 편의 기능을 추가해서 `ObjectProvider`가 만들어진 것이다. 

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;
public int logic() {
	PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
	prototypeBean.addCount();
	int count = prototypeBean.getCount();
	return count;
}
```

**실행 결과**

<img src="/images/2023-09-08-Spring_principle9/image-20230908194400166.png" alt="image-20230908194400166" style="zoom:80%;" />

- `prototypeBeanProvider.getObject()`를 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있다.
- `ObjectProvider`의 `getObject()`를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다.**(DL)**
- `ObjectProvider`는 지금 딱 필요한 DL 정도의 기능만 제공한다.



<br>

### JSR-330 Provider

두 번째 방법은 `javax.inject.Provider`라는 JSR-330 자바 표준을 사용하는 방법이다. 

이 방법을 사용하려면 `javax.inject:javax.inject:1` 라이브러리를 gradle에 추가해야 한다.(스프링부트 3.0 미만)

```java
@Autowired
private Provider<PrototypeBean> provider;
public int logic() {
	PrototypeBean prototypeBean = provider.get();
	prototypeBean.addCount();
	int count = prototypeBean.getCount();
	return count;
}
```

**실행 결과**

![image-20230909005237302](/images/2023-09-08-Spring_principle9/image-20230909005237302.png)

<br>

**특징**

- `get()`메서드 하나로 기능이 매우 단순하다.
- 별도의 라이브러리가 필요하다.
- 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.

<br>

**정리**

- 프로토타입 빈은, 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사용하면 된다. 그런데 실무에서는 대부분 싱글톤 빈으로 문제를 해결할 수 있기 때문에 프로토타입 빈을 직접적으로 사용하는 일은 드물다.
- `ObjectProvider`, `JSR330 Provider`등은 프로토타입 뿐만 아니라 DL이 필요한 경우 언제든지 사용할 수 있다.

<br>

## 웹 스코프

**웹 스코프의 특징**

- 웹 스코프는 웹 환경에서만 동작한다.
- 웹 스코프는 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리한다. 따라서 종료 메서드가 호출도니다.

<br>

**웹 스코프 종류**

- **request** : HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
- **session** : HTTP Session과 동일한 생명주기를 가지는 스코프
- **application** : 서블릿 컨텍스트와 동일한 생명주기를 가지는 스코프
- **websocket** : 웹 소켓과 동일한 생명주기를 가지는 스코프



<br>

**HTTP request 요청 당 각각 할당되는 request 스코프**

![image-20230910152219965](/images/2023-09-08-Spring_principle9/image-20230910152219965.png)



<br>

웹 스코프는 웹 환경에서만 동작하므로 웹 환경이 동작하도록 라이브러리를 추가하였다.

**build.gradle**

```java
//web 라이브러리 추가
implementation 'org.springframework.boot:spring-boot-starter-web'
```

<br>



### request 스코프 예제 개발

다음과 같은 로그가 남도록  request 스코프를 활용해 추가 기능을 개발해보자.

```
[d06b992f...] request scope bean create
[d06b992f...][http://localhost:8080/log-demo] controller test
[d06b992f...][http://localhost:8080/log-demo] service id = testId
[d06b992f...] request scope bean close
```

- 기대하는 공통 포맷 : [UUID] [requestURL] {message}
- UUID를 사용해서 HTTP 요청을 구분하자.
- requestURL 정보도 추가로 넣어서 어떤 URL을 요청해서 남은 로그인지 확인하자.



<br>

**MyLogger**

```java
package hello.core.common;
// import문 생략

@Component
@Scope(value = "request")
public class MyLogger {
  
	private String uuid;
	private String requestURL;
  
	public void setRequestURL(String requestURL) {
	this.requestURL = requestURL;
}
  
	public void log(String message) {
	System.out.println("[" + uuid + "]" + "[" + requestURL + "] " +
message);
	}
  
	@PostConstruct
	public void init() {
	uuid = UUID.randomUUID().toString();
	System.out.println("[" + uuid + "] request scope bean create:" + this);
	}
  
	@PreDestroy
	public void close() {
	System.out.println("[" + uuid + "] request scope bean close:" + this);
	}
}
로그를 출력하기 위한
```

- 로그를 출력하기 위한 `MyLogger` 클래스이다.
- `@Scope(value = "request")`를 사용해서 request스코프로 지정하였다. 이제 이 빈은 HTTP 요청 당 하나씩 생성되고, HTTP 요청이 끝나는 시점에 소멸된다.
- 이 빈이 생성되는 시점에 자동으로 `@PostConstruct`초기화 메서드를 사용하여 uuid를 생성하고 저장해준다. 이 빈은 HTTP 요청 당 하나씩 생성되므로, uuid를 저장해두면 다른 HTTP 요청과 구분할 수 있다.
- 이 빈이 소멸되는 시점에 `@PreDestroy`를 사용해서 종료 메시지를 남긴다.
- `requestURL`은 이 빈이 생성되는 시점에는 알 수 없으므로, 외부에서 setter로 입력받는다.



<br>

**LogDemoController**

```java
package hello.core.web;
// import문 생략

@Controller
@RequiredArgsConstructor
public class LogDemoController {
  
	private final LogDemoService logDemoService;
	private final MyLogger myLogger;
  
	@RequestMapping("log-demo")
	@ResponseBody
	public String logDemo(HttpServletRequest request) {
	String requestURL = request.getRequestURL().toString();
	myLogger.setRequestURL(requestURL);
   
	myLogger.log("controller test");
	logDemoService.logic("testId");
	return "OK";
	}
}
```

- 로거가 잘 작동하는지 확인하는 테스트용 컨트롤러이다.
- HttpServletRequest를 통해 요청 URL을 받았다. (requestURL 값 : `http://localhost:8080/log-demo`)
- 이렇게 받은 requestURL 값을 myLogger에 저장해준다. myLogger는 HTTP 요청 당 각각 구분되므로 다른 HTTP 요청 때문에 값이 섞이는 걱정은 하지 않아도 된다.
- 컨트롤러에서 controller test라는 로그를 남긴다.

<br>

**LogDemoService**

```java
package hello.core.logdemo;
import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LogDemoService {
  
	private final MyLogger myLogger;
  
	public void logic(String id) {
	myLogger.log("service id = " + id);
	}
}
```

- 비즈니스 로직이 있는 서비스 계층에서도 로그를 출력해보자.
- request scope를 사용하지 않고 MyLogger 덕분에 모든 정보를 파라미터로 넘기지 않고, MyLogger 멤버변수에 저장해서 코드와 계층을 깔끔하게 유지할 수 있다.

<br>

이제 실행을 해보려 했으나 오류가 발생한다.

```
Error creating bean with name 'myLogger': Scope 'request' is not active for the
current thread; consider defining a scoped proxy for this bean if you intend to
refer to it from a singleton;
```

스프링 애플리케이션은 실행하는 시점에 싱글톤 빈은 생성해서 주입이 가능하지만, request 스코프 빈은 아직 생성되지 않는다. 이 빈은 실제 고객의 요청이 와야 생성할 수 있다.



<br>

### 첫 번째 해결방법 - ObjectProvider 사용하기

**LogDemoController**

```java
package hello.core.web;

@Controller
@RequiredArgsConstructor
public class LogDemoController {
  
	private final LogDemoService logDemoService;
	private final ObjectProvider<MyLogger> myLoggerProvider;
  
	@RequestMapping("log-demo")
	@ResponseBody
	public String logDemo(HttpServletRequest request) {
	String requestURL = request.getRequestURL().toString();
	MyLogger myLogger = myLoggerProvider.getObject();
 	myLogger.setRequestURL(requestURL);
 
	myLogger.log("controller test");
	logDemoService.logic("testId");
	return "OK";
	}
}
```

<br>

**LogDemoService**

```java
package hello.core.logdemo;

@Service
@RequiredArgsConstructor
public class LogDemoService {
  
	private final ObjectProvider<MyLogger> myLoggerProvider;
  
	public void logic(String id) {
	MyLogger myLogger = myLoggerProvider.getObject();
	myLogger.log("service id = " + id);
	}
}
```

ObjectProvider를 적용한 후 `http:localhost:8080/log-demo`로 접속하였더니 잘 작동하게 되었다.

```
[d06b992f...] request scope bean create
[d06b992f...][http://localhost:8080/log-demo] controller test
[d06b992f...][http://localhost:8080/log-demo] service id = testId
[d06b992f...] request scope bean close
```

- `ObjectProvider`덕분에 `ObjectProvider.getObject()`를 호출하는 시점까지 request scope의 **빈이 생성을 지연**할 수 있었다.
- `ObjectProvider.getObject()`를 호출하는 시점에는 HTTP 요청이 진행중이므로  request scope 빈의 생성이 정상 처리된다.
- `ObjectPRovider.getObject()`를 `LogDemoController`, `LogDemoService`에서 각각 한번씩 따로 호출해도 같은 HTTP 요청이면 같은 스프링 빈이 반환되는데, 이걸 직접 구분하려면 상당히 힘들다.



<br>

### 스코프와 프록시

이번에는 프록시 방법을 사용해보자.

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
}
```

- `proxyMode = ScopedProxyMode.TARGET_CLASS`를 추가해주자.
    - 적용 대상이 클래스면 `TARGET_CLASS`를 선택, 인터페이스면 `INTERFACES`를 선택
- 이렇게 하면 MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입해 둘 수 있다.

이렇게 하고 나머지 코드를 Provider 사용 이전으로 돌려두고 실행하면 잘 동작하게 된다.



**웹 스코프와 프록시 동작 원리**

먼저 주입된 MyLogger를 확인해보자.

```java
System.out.println("myLogger = " + myLogger.getClass());
```

<br>

**출력결과**

```java
myLogger = class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$$54997f82
```

**CGLIB라는 라이브버리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.**

- `@Scope` 의 `proxyMode = ScopedProxyMode.TARGET_CLASS)` 를 설정하면 스프링 컨테이너는 CGLIB 라는 바이트코드를 조작하는 라이브러리를 사용해서, MyLogger를 상속받은 가짜 프록시 객체를 생성한다

- 결과를 확인해보면 우리가 등록한 순수한 MyLogger 클래스가 아니라 `MyLogger$ $EnhancerBySpringCGLIB` 이라는 클래스로 만들어진 객체가 대신 등록된 것을 확인할 수 있다
- 그리고 스프링 컨테이너에 "myLogger"라는 이름으로 진짜 대신에 이 가짜 프록시 객체를 등록한다
- `ac.getBean("myLogger", MyLogger.class)` 로 조회해도 프록시 객체가 조회되는 것을 확인할 수 있다.
- 그래서 의존관계 주입도 이 까자 프록시 객체가 주입된다.

<br>

![image-20230910154714035](/images/2023-09-08-Spring_principle9/image-20230910154714035.png)

**가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다.**

- 가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 방법을 알고 있다.
-  클라이언트가 `myLogger.logic()` 을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것이다. 
- 가짜 프록시 객체는 request 스코프의 진짜 `myLogger.logic()` 를 호출한다. 
- 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 사실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)

<br>

**동작 정리**

- CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다. 
- 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있다. 
- 가짜 프록시 객체는 실제 request scope와는 관계가 없다. 그냥 가짜이고, 내부에 단순한 위임 로직만 있고, 싱글톤 처럼 동작한다.

<br>

**특징 정리**

- 프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request scope를 사용할 수 있다. 
- 사실 Provider를 사용하든, 프록시를 사용하든 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연처리 한다는 점이다. 
- 단지 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다. 이것이 바로 다형성과 DI 컨테이너가 가진 큰 강점이다. 
- 꼭 웹 스코프가 아니어도 프록시는 사용할 수 있다

<br>

**주의점**

- 마치 싱글톤을 사용하는 것 같지만 다르게 동작하기 때문에 결국 주의해서 사용해야 한다. 
- 이런 특별한 scope는 꼭 필요한 곳에만 최소화해서 사용하자, 무분별하게 사용하면 유지보수하기 어려워진다

