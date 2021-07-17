
@Transactional에 대한 질문을 받았다.
하지만 대답하지 못했다. 이것을 제대로 알기 위해서는 AOP를 알아야 대답을 할 수 있다는 것을 느꼈고 AOP에 대해 생각을 해보았으나, 정확한 대답을 하지 못했다.
즉, 잘 모른다는 것이다. 그래서 이번 기회에 제대로 정리해보고 트랜잭션까지 공부해봐야겠다.

## 1. AOP 개념

Asptect-oriented Programming(AOP)은 OOP를 보완하는 수단으로 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법이다. (OOP를 더 잘할 수 있게 도와준다.)

![](https://i.imgur.com/BZfa6OV.png)

동일한 색깔은 동일한 concern이라고 생각하면 된다. Concern이란 여러 클래스, 메소드에 거쳐 나타나는 비슷한 코드들이다.
ex) transaction, logging

각 concern의 변경이 일어나면? 유지보수가 쉽지 않다.

AOP는 이 문제를 Aspect를 이용해서 해결한다. 흩어진 것들을 한군데로 모아서 각각의 concern 별로 Aspect를 만든다. Aspect 안에 실제 하던 일과 어디에 적용해야 하는지를 독립적으로 정의한다. 즉, 해야 할 일과 그 일을 어디에 적용해야 하는지 묶어서 모듈화하는 것이 AOP이다.

### 1-1. AOP 용어 설명

* Aspect: 동일한 concern을 묶어 모듈화 한 것이다. Aspect 안에는 Advice와 Point Cut이 들어간다.
* Advice: 해야 할 일들을 말한다.
* Point Cut: 어디에 적용해야 하는지에 대한 정보이다.
* Target: 적용이 되는 대상이다.
* Join Point: 적용할 수 있는 지점을 말한다. 구체적이다. ex) 생성자 호출하기 직전, 호출했을 때, 필드에 접근하기 전, 필드에서 값을 가져갔을 때

이렇게 보면 이해하기 쉽지 않다. 코드를 보며 이해해보자.

```java=
class HelloService {

    public void foo() {
         System.out.println("foo입니다.");
    }
}
```

HelloServie의 foo() 메소드 실행 전, 후에 hello랑 bye를 출력하는 일을 한다고 가정하자. 이때 "HelloService의 Bean"이 **target**, "foo() 메소드 실행 전, 후"가 **Point Cut**, "메소드 실행 전, 후"가 **Join Point**, "hello와 bye를 출력하는 일"이 **Advice**이다.

Point Cut과 Join Point가 헷갈리는데 Join Point는 메타적인 정보이고 Point Cut은 좀 더 구체적인 적용 지점이라고 생각할 수 있다.

### 1-2. AOP 적용 방법

```
A 클래스에 foo() 메소드가 있다.
그리고 Hello를 출력하는 Aspect가 있다.
여기서 foo()를 호출하면 Hello가 먼저 출력이 되어야 한다.
```

위와 같은 과정에서 어떻게 AOP가 적용되는지 확인해보자.

#### 1) 컴파일 시점

.java 파일을 .class 파일로 만들 때, 바이트 코드들을 조작하여 생성해낸다.

로드 타임, 런타임 때 성능적인 부하가 없다. 하지만 별도의 컴파일 과정을 거쳐야 한다.

#### 2) 로드 타임 시점

A 클래스는 순수한 클래스로 컴파일이 된다. foo() 메소드도 순수한 기능이 정의된다.
A 클래스 파일을 로딩하는 시점에 로딩하는 클래스 정보를 변경한다.

A 클래스의 바이트 코드는 순수하게 있지만, JVM이 로딩하면서 JVM 메모리상에 foo() 메소드를 호출하기 전 Hello를 출력하는 Aspect를 포함한 상태로 로딩을 한다.

클래스 로딩할 때 부하가 생길 수 있고 로드 타임 위버를 설정해줘야 한다.

#### 3) 런타임 시점

스프링 AOP가 사용하는 방법이다.

스프링 환경이기 때문에 A 클래스는 Bean이 된다. A라는 Bean에 Aspect에 적용해야 함을 스프링이 알고 있다.

A 클래스의 빈을 만들 때, A 클래스의 프록시 빈을 먼저 만든다. 프록시 빈이 A 클래스가 가지고 있는 foo() 메소드를 실행하기 직전에 Hello를 먼저 출력하고 foo() 메소드를 호출한다.

AOP를 위한 설정이 필요하지 않고 별도의 컴파일 또한 필요 없다. 하지만 빈을 만드는 초기에 성능을 잡아먹는다.


> AspectJ: 컴파일, 로드 타임
> Spring AOP: 런타임

### 1-3. AOP 구현체

https://ko.wikipedia.org/wiki/%EA%B4%80%EC%A0%90_%EC%A7%80%ED%96%A5_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D#%EA%B5%AC%ED%98%84

* AspectJ
* 스프링 AOP


## 2. 프록시 기반 AOP

스프링 AOP는 프록시 기반 AOP 구현체이다. 스프링 빈에만 AOP를 적용할 수 있다.

### 2-1. 프록시 패턴

![](https://i.imgur.com/w6OEzRV.png)

프록시 패턴은 접근 제어 또는 부가 기능 추가의 목적으로 사용된다.

### 2-2. 구현하며 이해하기

#### Interface
```java=
public interface EventService {

    void createEvent();

    void publishEvent();

    void deleteEvent();
}
```

#### RealSubject
```java=
@Service
public class RealEventService implements EventService {

    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Creat Event");
    }

    @Override
    public void publishEvent() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Publish Event");
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```

#### Client

```java=
@Component
public class AppRuner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent();
        eventService.publishEvent();
    }
}
```

위 코드에 메소드 시간을 측정하는 기능을 추가해보자.

먼저, Real Subject에 시간 측정 코드를 추가해보자.

#### Real Subject
```java=
@Service
public class RealEventService implements EventService {

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Creat Event");
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Publish Event");
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```

메소드의 시간을 추가하였지만, 기존의 코드를 건드리고 있다. 이렇게 되면 메소드는 실질적인 기능 외에도 구현되어 있기 때문에 SRP에 어긋난다고 할 수 있다. 또한, 시간을 측정하고자 하는 메소드마다 중복된 코드를 추가해야 하므로 비효율적이다.

이를 어떻게 해결할 수 있을까?

우선 기존 코드를 원상복구 시켜보자.

#### Real Subject

```java=
@Service
public class RealEventService implements EventService {

    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Creat Event");
    }

    @Override
    public void publishEvent() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Publish Event");
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```

### 2.3 프록시 방법

위 코드에 프록시 패턴을 적용해보자.

```java=
@Primary
@Service
public class ProxyRealEventService implements EventService{

    @Autowired
    EventService realEventService; 

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis(); 
        realEventService.createEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis(); 
        realEventService.publishEvent();
        System.out.println(System.currentTimeMillis() - begin); 
    }

    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```

기존 코드를 감싸주는 프록시 패턴을 적용함으로써 우리는 메소드의 시간을 측정할 수 있게 되었다.

하지만 이 코드에서도 중복된 코드가 나타난다. 어떻게 이 문제를 해결할 수 있을까?

## 3. @AOP

스프링 AOP를 사용하면 위의 문제를 해결할 수 있다.

### 3-1. AOP 의존성 추가

#### gradle

```groovy=
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

#### maven

```xml=
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 3-2. Aspect 사용

@Aspect 어노테이션을 사용하면 해당 클래스가 Aspect임을 명시해준다.

```java=
@Component
@Aspect
public class PerfAspect {
    ...
}
```

Aspect에 Adivce와 PointCut이 정의하자.

#### 1) Adivce 정의

```java=
public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
    long begin = System.currentTimeMillis();
    Object retVal = proceedingJoinPoint.proceed();
    System.out.println(System.currentTimeMillis() - begin);
    return retVal;
}
```

ProceedingJoinPoint는 이 advice가 적용이 되는 대상이다. (createEvent, publishEvent 메소드 자체)

Advice는 3가지 방법으로 적용시킬 수 있다.

* Before
    * Advice는 메소드가 실행되기 전에 동작한다.
* After
    * Advice는 메소드가 실행된 후에 동작한다.
* Around
    * Before + After이라고 생각하면 된다. 즉, 메소드 실행 전, 후에 동작한다.

#### 2) PointCut 정의

PointCut은 3가지 방법으로 적용시킬 수 있다.

* execution
    * 포인트 컷 표현식을 사용하여 어디에 적용할 것인지 정의할 수 있다.
* annotaion

아래와 같이 annotation을 정의한다.
```java=
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.METHOD)
public @interface PerfLogging {
}
```

이후 @annotation 표현식을 통해 정의해줄 수 있다.
그렇게 되면 PerfLogging Annotation을 사용한 메소드들에게 advice를 적용해준다.

```java=
@Around("@annotation(PerfLogging)")
public Object logPerf(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
    long begin = System.currentTimeMillis();
    Object retVal = proceedingJoinPoint.proceed();
    System.out.println(System.currentTimeMillis() - begin);
    return retVal;
```
* bean

@bean 표현식을 통해 정의해줄 수 있다.

```java=
@Around("@bean(realEventService)")
public Object logPerf(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
    long begin = System.currentTimeMillis();
    Object retVal = proceedingJoinPoint.proceed();
    System.out.println(System.currentTimeMillis() - begin);
    return retVal;
```

#### 참고

https://engkimbs.tistory.com/746

https://www.inflearn.com/course/spring-framework_core
