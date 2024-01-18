# 들어가며

---

해당 포스트에서는 `Bean Scope`에 대해서 공부하고 정리하는 곳 입니다.

# 빈 스코프란?

---

`Bean`은 스프링에서 사용하는 `POJO`기반 객체입니다.

상황과 필요에 따라 `Bean`을 사용할 때 하나만 만들어야할 수도 있고, 여러개가 필요할 때도 있고, 어떤 한 시점에서만 사용해야할 때가 있을 수 있습니다.
이를 위해 `Scope`를 설정해서 `Bean`의 사용 범위를 개발자가 설정할 수 있습니다.

> 💫 Info
> 우선 따로 설정해주지 않으면, `Spring` 에서 `Bean` 은 싱글톤으로 생성됩니다.

## 빈 스코프 종류

---

스프링은 다음과 다양한 스코프를 지원합니다.

- 싱글톤 : 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프입니다.
- 프로토타입 : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프입니다.
- 웹 관련 스코프 : 
	- `request` : 웹 요청이 들어오고 나갈때 까지 유지되는 스코프입니다.
	- `session` : 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프입니다.
	- `application` : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프입니다.

## 빈 스코프 등록

---

빈 스코프는 다음과 같이 등록할 수 있습니다.

**컴포넌트 스캔 자동 등록**

```java
@Scope("prototype")  
@Component  
public class HelloBean {  
}
```

**수동 등록**

```java
@Scope("prototype")  
@Bean
PrototypeBean HelloBean {  
	return new HelloBean();
}
```

## 싱클톤 빈 요청

---

먼저 싱글톤의 빈 요청을 확인해보겠습니다.

<img width="700" alt="2023-10-18_10-21-56" src="https://github-production-user-asset-6210df.s3.amazonaws.com/74192619/276047697-77545b4e-f6b3-4708-a305-df878a6586dd.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231018%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231018T012843Z&X-Amz-Expires=300&X-Amz-Signature=087fbb73d0a460f2071915ef4e4983b321d19ba5fa5f114afecf742a1eb94f37&X-Amz-SignedHeaders=host&actor_id=74192619&key_id=0&repo_id=693917725">

1. 싱글톤 스코프의 빈을 스프링 컨테이너에 요청합니다.
2. 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환합니다.
3. 이후에 스프링 컨테이너에 같은 요청이 와도 같은 객체 인스턴스의 스프링 빈을 반환합니다.

### 예제 코드

---

싱글톤 스코프 빈을 테스트 해봅니다.


```java
@Slf4j  
class SingletonTest {  
    @Test  
    void singletonBeanTest() {  
	    /* Given */  
	    AnnotationConfigApplicationContext ac = new 
		    AnnotationConfigApplicationContext(SingletonBean.class);  
	    SingletonBean bean1 = ac.getBean(SingletonBean.class);  
	    SingletonBean bean2 = ac.getBean(SingletonBean.class);  
	  
	    /* When */  
	    log.info("SingletonBean1 : [{}]", bean1);  
	    log.info("SingletonBean2 : [{}]", bean2);  
	  
	    /* Then */  
	    assertThat(bean1).isSameAs(bean2);  
    }  
  
    @Scope("singleton")  
    static class SingletonBean {  
	    @PostConstruct  
	    public void init() {  
		    System.out.println("SingletonBean.init");  
	    }  
	  
	    @PreDestroy  
	    public void destroy() {  
		    System.out.println("SingletonBean.destroy");  
	    }  
    }  
}
```

다음과 같이 코드를 작성하여 실행된 결과는 다음과 같습니다.

```text
[Test worker] INFO com.study.core.basic.beanscope.SingletonTest -- SingletonBean.init
[Test worker] INFO com.study.core.basic.beanscope.SingletonTest -- SingletonBean1 : [com.study.core.basic.beanscope.SingletonTest$SingletonBean@5a85c92]
[Test worker] INFO com.study.core.basic.beanscope.SingletonTest -- SingletonBean2 : [com.study.core.basic.beanscope.SingletonTest$SingletonBean@5a85c92]
[Test worker] INFO com.study.core.basic.beanscope.SingletonTest -- SingletonBean.destroy
```

- 빈 초기화 메서드를 실행하였습니다. (`init`)
- 같은 인스턴스의 빈을 조회하였습니다. (예제 코드의 `isSameAs` 가 성공함)
- 종료 메서드까지 정상 호출 되었습니다. (`destroy`)

## 프로토타입 빈 요청

---

위에서 싱글톤 스코프에 대한 빈 요청을 확인해 보았습니다.

이번에는 프로토타입의 빈을 요청하는 흐름을 확인하도록 하겠습니다.

<img width="700" alt="2023-10-18_10-36-16" src="https://github-production-user-asset-6210df.s3.amazonaws.com/74192619/276048912-7b3c2a91-904d-4778-9993-30b0de9668aa.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231018%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231018T014138Z&X-Amz-Expires=300&X-Amz-Signature=22ad4c4f2fca229623772ae9aa3f3071b0d71f63c89a7a8bb3df02692dd3bf9e&X-Amz-SignedHeaders=host&actor_id=74192619&key_id=0&repo_id=693917725">

1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청합니다.
2. 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입합니다.

> 💫 Info
> 필요한 의존관계를 주입하는 것은 초기화 메서드까지 호출하는 것을 뜻합니다.

<img width="700" alt="2023-10-18_10-36-28" src="https://github-production-user-asset-6210df.s3.amazonaws.com/74192619/276048920-074b22ea-6c8d-4525-a96a-579fa29de6bb.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A/20231018/us-east-1/s3/aws4_request&X-Amz-Date=20231018T014159Z&X-Amz-Expires=300&X-Amz-Signature=5748f18c952b9c2ae75d8a4756a03786e216c51bab548e96bdf440fbde8a7306&X-Amz-SignedHeaders=host&actor_id=74192619&key_id=0&repo_id=693917725">

3. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환합니다.
4. 이후에 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환합니다.

### 정리

---

**핵심은 스프링 컨테이너에 프로토타입 빈을 생성하고, 의존관계 주입 및 초기화까지만 처리하는 것**입니다. 클라이언트에 빈을 반환하고, 이후 스프링 컨테이너는 생성된 프로토타입 빈을 관리하지 않습니다. **프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있습니다. 그래서 `@PreDestroy` 같은 종료 메서드가 호출됩니다.**

> 💫 Info
> `@PreDestroy` 는 자바 표준의 어노테이션입니다.
> 스프링에서도 해당 어노테이션을 적극 사용하는 것을 권하고 있습니다.

### 예제 코드

---


```java
@Slf4j  
class PrototypeTest {  
    @Test  
    public void prototypeBeanFind() {  
	    AnnotationConfigApplicationContext ac =  
		    new AnnotationConfigApplicationContext(PrototypeBean.class);  
  
	    log.info("find PrototypeBean1");  
	    PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);  
	    log.info("find PrototypeBean2");  
	    PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);  
  
	    log.info("PrototypeBean1 : [{}]", prototypeBean1);  
	    log.info("PrototypeBean2 : [{}]", prototypeBean2);  
  
	    assertThat(prototypeBean1).isNotSameAs(prototypeBean2);  
	    ac.close();
    }  
  
    @Scope("prototype")  
    static class PrototypeBean {  
    @PostConstruct  
	    public void init() {  
		    System.out.println("PrototypeBean.init");  
	    }  
  
    @PreDestroy  
	    public void destroy() {  
		    System.out.println("PrototypeBean.인스턴스 이니셜라이저");  
	    }  
    }  
}
```

실행 결과는 다음과 같습니다.

```text
[Test worker] INFO com.study.core.basic.beanscope.PrototypeTest -- find PrototypeBean1
PrototypeBean.init
[Test worker] INFO com.study.core.basic.beanscope.PrototypeTest -- find PrototypeBean2
PrototypeBean.init
[Test worker] INFO com.study.core.basic.beanscope.PrototypeTest -- PrototypeBean1 : [com.study.core.basic.beanscope.PrototypeTest$PrototypeBean@62163b39]
[Test worker] INFO com.study.core.basic.beanscope.PrototypeTest -- PrototypeBean2 : [com.study.core.basic.beanscope.PrototypeTest$PrototypeBean@4504d271]
```

- 싱글톤과 다르게 `@PreDestory`가 실행되지 않았습니다.
### 정리

---

- 스프링 컨테이너에 요청할 때 마다 새로 생성됩니다.
- 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입, 초기화까지만 관여합니다.
- 종료 메서드가 호출되지 않아서 클라이언트에서 처리해야 합니다.
  
> 💫 Info
> 대부분의 싱글톤과 소수의 프로토타입을 사용합니다.

# Ref

---

1. [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)
2. [[Spring] Bean Scope](https://github.com/gyoogle/tech-interview-for-developer/blob/master/Web/Spring/%5BSpring%5D%20Bean%20Scope.md#spring-bean-scope)
