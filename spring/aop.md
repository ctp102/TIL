# AOP
## JDK 동적 프록시
- 인터페이스

## CGLIB
- 구체 클래스

## 프록시 팩토리
```java
@Slf4j
public class ProxyFactoryTest {
    @Test
    @DisplayName("인터페이스가 있으면 JDK 동적 프록시 사용")
    void interfaceProxy() {
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());
        ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        Assertions.assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        Assertions.assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue(); // 프록시 팩토리로 생성한 JDK 동적프록시인지 체크(직접 만든 JDK 동적 프록시는 false)
        Assertions.assertThat(AopUtils.isCglibProxy(proxy)).isFalse();
    }
}
```
```java
@Slf4j
public class ProxyFactoryTest {
    @Test
    @DisplayName("구체 클래스만 있으면 CGLIB 프록시 사용")
    void concreteProxy() {
        ConcreteService target = new ConcreteService();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());
        ConcreteService proxy = (ConcreteService) proxyFactory.getProxy();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        Assertions.assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        Assertions.assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse(); // 프록시 팩토리로 생성한 JDK 동적프록시인지 체크(직접 만든 JDK 동적 프록시는 false)
        Assertions.assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
    }
}
```
```java
@Slf4j
public class ProxyFactoryTest {
    @Test
    @DisplayName("proxyTargetClass 옵션을 사용하면 인터페이스가 있어도 CGLIB 프록시 사용")
    void proxyTargetClass() {
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);

        // 아래 옵션을 넣으면 target이 인터페이스여도 CGLIB 프록시를 만들어준다.
        proxyFactory.setProxyTargetClass(true);

        proxyFactory.addAdvice(new TimeAdvice());
        ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        Assertions.assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        Assertions.assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse(); // 프록시 팩토리로 생성한 JDK 동적프록시인지 체크(직접 만든 JDK 동적 프록시는 false)
        Assertions.assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
    }
}
```

- 스프링부트 2.0부터 기본적으로 AOP를 사용할 때 proxyFactory.setProxyTargetClass(true); 옵션을 주어 인터페이스가 있어도 CGLIB 프록시를 사용한다.