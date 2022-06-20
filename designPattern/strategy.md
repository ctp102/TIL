## 전략 패턴

- 공통적인 부분은 Context 클래스로, 바뀌는 부분은 Strategy 인터페이스에 구현된 추상메서드를 구현체로 구현한다.
- Context 클래스를 사용하는 방법은 두 가지가 있다.
    1. Strategy를 필드에 보관하는 방식
    2. Strategy를 파라미터로 전달 받는 방식(템플릿 콜백 패턴이라고 보면 된다.)

### 1. Strategy를 필드에 보관하는 방식
```java
public interface Strategy {

    void call();

}
```

```java

@Slf4j
public class StrategyLogic1 implements Strategy {

    @Override
    public void call() {
        log.info("비즈니스 로직 실행1");
    }
}
```

```java
/**
 * 전략을 필드에 보관하는 방식
 */
@Slf4j
public class ContextV1 {

    private Strategy strategy;

    public ContextV1(Strategy strategy) {
        this.strategy = strategy;
    }

    public void execute() {
        long startTime = System.currentTimeMillis();

        // 비즈니스 로직 실행
        strategy.call(); // 위임

        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
}
```
```java

@Slf4j
public class ContextV1Test {

    /**
     * 전략 패턴 사용
     */
    @Test
    void strategyV1() {
        Strategy strategyLogic1 = new StrategyLogic1();
        ContextV1 context1 = new ContextV1(strategyLogic1);
        context1.execute();

        Strategy strategyLogic2 = new StrategyLogic2();
        ContextV1 context2 = new ContextV1(strategyLogic2);
        context2.execute();
    }

    /**
     * 익명 내부 클래스 사용
     */
    @Test
    void strategyV2() {

        ContextV1 context1 = new ContextV1(new Strategy() {
            @Override
            public void call() {
                log.info("비즈니스 로직 실행1");
            }
        });
        context1.execute();

        ContextV1 context2 = new ContextV1(new Strategy() {
            @Override
            public void call() {
                log.info("비즈니스 로직 실행2");
            }
        });
        context2.execute();
    }

    /**
     * 람다 사용
     */
    @Test
    void strategyV3() {

        ContextV1 context1 = new ContextV1(() -> log.info("비즈니스 로직 실행1"));
        context1.execute();

        ContextV1 context2 = new ContextV1(() -> log.info("비즈니스 로직 실행2"));
        context2.execute();
    }

}
```

### 2. Strategy를 파라미터로 전달 받는 방식
```java
/**
 * 전략을 파라미터로 전달 받는 방식
 */
@Slf4j
public class ContextV2 {

    public void execute(Strategy strategy) {
        long startTime = System.currentTimeMillis();

        // 비즈니스 로직 실행
        strategy.call(); // 위임

        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
}

```
```java
@Slf4j
public class ContextV2Test {

    @Test
    void strategyV1() {
        ContextV2 context = new ContextV2();
        context.execute(new StrategyLogic1());
        context.execute(new StrategyLogic2());
    }

    @Test
    void strategyV2() {
        ContextV2 context = new ContextV2();
        context.execute(new Strategy() {
            @Override
            public void call() {
                log.info("비즈니스 로직 실행1");
            }
        });
        context.execute(new Strategy() {
            @Override
            public void call() {
                log.info("비즈니스 로직 실행2");
            }
        });
    }

    @Test
    void strategyV3() {
        ContextV2 context = new ContextV2();
        context.execute(() -> log.info("비즈니스 로직 실행1"));
        context.execute(() -> log.info("비즈니스 로직 실행2"));
    }

}
```