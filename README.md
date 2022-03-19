# DesignPattern_Study_Strategy

# Notion Link
https://five-cosmos-fb9.notion.site/Strategy-35eb795d671e40ed9af6b18ecd66ddad


# 전략 (Strategy)

### 의도

알고리즘 제품군을 정의하고 **각각을 캡슐화하여** 상호 교환 가능하게 만들자.

전략을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.

![image](https://user-images.githubusercontent.com/18654358/159111845-4767bd43-bec6-49f0-8498-e99634dec774.png)

### 탬플릿 메서드 패턴과의 비교

**탬플릿 메서드 패턴**

부모 클래스에 변하지 않는 템플릿을 두고, 변하는 부분을 자식 클래스에 두어서 **“상속”**을 사용해 문제를 해결하는 패턴이다.

**전략 패턴**

**변하지 않는 부분**을 Context 라는 곳에 두고,

**변하는 부분**을 Stragety 라는 **`인터페이스`**를 만들고 해당 인터페이스를 구현하도록 해서 문제를 해결한다.

<aside>
🎈 즉, 전략 패턴은 상속이 아니라 위임으로 문제를 해결하는 것이다.

</aside>

- 전략 패턴에서 Context : 변하지 않는 템플릿의 역할
- 전랙 패턴에서 Strategy : 변하는 알고리즘 역할

**Strategy를 구성해보자.**

[Strategy.java](http://Strategy.java) 인터페이스 정의

→ 이 전략 인터페이스는 **`“변하는 알고리즘”`** 역할을 한다.

```java
package hello.advanced.trace.strategy.code.strategy;

public interface Strategy {
    void call();
}
```

[Strategy.java](http://Strategy.java) 인터페이스를 구현한 StrategyLogic1.java, StrategyLogic2.java

```java
package hello.advanced.trace.strategy.code.strategy;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class StrategyLogic1 implements Strategy {
    @Override
    public void call() {
        log.info("비즈니스 로직1 실행");
    }
}

///////////////////////////////////////////////////////////
package hello.advanced.trace.strategy.code.strategy;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class StrategyLogic2 implements Strategy {
    @Override
    public void call() {
        log.info("비즈니스 로직2 실행");
    }
}
```

**Context를 구성해보자.**

[ContextV1.java](http://ContextV1.java) 클래스 정의

- ContextV1 은 **변하지 않는 로직**을 가지고 있는 **템플릿 역할**을 하는 코드이다.
- 전략 패턴에서 이것을 **문맥 컨텍스트** 라고 한다.
    - 컨텍스트(문맥)은 변하지 않지만, 그 문맥 속에서 전략(Strategy)를 통해 일부 전략이 변경된다고 보면 된다.
- Context 내부에는 **Strategy strategy** 필드를 가지고 있다.
    - 이 필드는 변하는 부분인 **strategy의 구현체를 주입**하면 된다.
- ***전략 패턴의 핵심은 Context는 Strategy 인터페이스에만 의존한다는 것이다.***
    - 덕분에 Strategy의 구현체를 변경하거나 새로 만들어도 Context 코드에는 영향이 없다.

<aside>
💡 어디서 많이 본 코드일 것이다!
- 스프링에서 의존관계 주입을 할 때 사용하는 방식이 바로 **“전.략.패.턴”** 이다.

</aside>

```java
package hello.advanced.trace.strategy.code.strategy;

import lombok.extern.slf4j.Slf4j;

/**
 * 필드에 전략을 보관하는 방식
 */
@Slf4j
public class ContextV1 {

    private Strategy strategy;

    public ContextV1(Strategy strategy){
        this.strategy = strategy;
    }

    public void execute(){
        long startTime = System.currentTimeMillis();

        // 비즈니스 로직 실행
        strategy.call();
        // 비즈니스 로직 종료

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime = {}", resultTime);
    }

}
```

**전략 패턴을 이용한 테스트 코드를 작성해보자.**

```java
/**
 * 전략 패턴 사용
 */
@Test
void strategyV1(){
    // 전략 생성
    StrategyLogic1 strategyLogic1 = new StrategyLogic1();

    // 컨텍스트 생성 + 전략 주입
    ContextV1 contextV1 = new ContextV1(strategyLogic1);

    // 컨텍스트 실행
    contextV1.execute();

    StrategyLogic2 strategyLogic2 = new StrategyLogic2();
    ContextV1 contextV1_2 = new ContextV1(strategyLogic2);
    contextV1_2.execute();
}
```

- 코드를 보면 의존관계 주입을 통해 ContextV1에 Strategy의 구현체인 StrategyLogic1을 주입하는 것을 볼 수 있다.
- 이렇게 해서 Context안에 원하는 전략을 주입한다.
- 이렇게 원하는 모양으로 조립을 완료하고 난 다음에 Context의 execute를 호출하여 실행한다.

**실행 그림**

![image](https://user-images.githubusercontent.com/18654358/159111853-f6b18088-cd29-449a-8028-b0512c222b09.png)

1. Context에 원하는 Strategy 구현체를 주입한다.
2. 클라이언트는 Context를 실행한다.
3. Context는 Context 로직을 시작한다.
4. Context로직 중간에 strategy.call()을 호출하여 주입받은 Strategy 로직을 실행한다.
5. Context 나머지 로직을 수행한다.

**전략 패턴도 “익명 내부 클래스”를 사용할 수 있다.**

```java
/**
 * 전략 패턴 사용 - 익명 내부 클래스 이용
 */
@Test
void strategyV2(){
		/*
			Strategy strategy = new Strategy() {
		      @Override
		      public void call() {
		          log.info("비즈니스 로직");
		      }
		  };
		*/
    Strategy strategyLogic1 = () -> log.info("비즈니스 로직 1 실행");
    ContextV1 contextV1 = new ContextV1(strategyLogic1);
    contextV1.execute();

    Strategy strategyLogic2 = () -> log.info("비즈니스 로직 2 실행");
    ContextV1 contextV1_2 = new ContextV1(strategyLogic2);
    contextV1_2.execute();
}
```

**익명 내부 클래스**를 조금 더 편하게 사용해보자.

```java
/**
 * 전략 패턴 사용 - 익명 내부 클래스 이용  
 *  변수로 만들어 사용하지 않고 바로 사용하기 with Lambda
 */
@Test
void startegyV3(){
    ContextV1 contextV1 = new ContextV1(() -> log.info("비즈니스 로직 1 실행"));
    contextV1.execute();

    ContextV1 contextV1_2 = new ContextV1(() -> log.info("비즈니스 로직 2 실행"));
    contextV1_2.execute();
}
```

<aside>
💡 Lambda 로 바꿀 수 있었던 이유? 인터페이스가 하나만 있기 때문!

</aside>

---

---

### 선 조립, 후 실행

여기서 이야기 하고 싶은 부분은 **Context의 내부 필드에 Strategy를 두고 사용하는 부분**이다.

이 방식은 Context와 Strategy를 **실행하기 전에 원하는 모양을 조립**해두고,

다음에 Context를 실행하는 선 조립, 후 실행 방식에서 매우 유용하다.

Context와 Strategy를 한번 조립하고 나면 이후로는 Context를 실행하기만 하면 된다.

<aside>
💡 우리가 스프링 애플리케이션을 개발할 때 로딩 시점에 의존관계 주입을 통해 필요한 의존관계를 모두 맺어두고 실제 요청 처리를 하는 것과 같은 원리이다.

</aside>

**`이 방식의 단점`**은 Context와 Strategy를 **조립한 이후에는 전략을 변경하기가 번거롭다**는 점이다.

물론 Context에 Setter를 제공하여 Strategy 구현체 를 변경하여 처리하면 되지만,

***Context를 `싱글톤`으로 사용할 때 동시성 이슈 등 고려할 점이 많아지게 된다.***

<aside>
💡 그래서 전략을 실시간으로 변경해야 한다면 ? setter로 주입하는 것 보다
차라리 이전에 개발한 테스트 코드 처럼 
Context를 하나 더 생성하고 그곳에 다른 Strategy를 주입하는 것이 더 나은 방법이 될 수 있다.

</aside>

***이렇게 먼저 조립하고 실행하는 방식보다***

***더 유연하게 전략 패턴을 사용할 수 있는 방법은 없을까?***

---

---

**전략 패턴을 조금 다르게 사용해보자**

Context의 필드에 Strategy를 주입해서 사용했었다. (선 조립 후 실행)

- ***실행하는 시점에 파라미터로 전달해서 사용해보자! 즉, 실행시점에 조립하여 실행하자!***

**ContextV2.java**

- 전략을 execute()가 실행 될 때마다 전달받아 사용하는 것!

```java
/**
 * 전략을 파라미터로 전달받는 방식 (전략패턴을 조금 변형!?)
 */
@Slf4j
public class ContextV2 {

    public void execute(Strategy strategy){
        long startTime = System.currentTimeMillis();

        // 비즈니스 로직 실행
        strategy.call();
        // 비즈니스 로직 종료

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime = {}", resultTime);
    }

}
```

**테스트 코드를 작성해보자**

```java
package hello.advanced.trace.strategy.code.strategy;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

@Slf4j
public class ContextV2Test {

    /**
     * 전략 패턴 적용
     */
    @Test
    void strategyV1(){
        ContextV2 context = new ContextV2();
        context.execute(() -> log.info("비즈니스 로직 1 실행"));
        context.execute(() -> log.info("비즈니스 로직 2 실행"));
    }
}
```

위와 같이 구성한다면?

Context와 Strategy를 **`“선 조립 후 실행” 하는 방식이 아니라`**

***Context를 실행하는 시점마다 전략을 파라미터로 전달한다.***

클라이언트는 Context를 실행하는 시점에 원하는 Strategy를 전달할 수 있다.

따라서 이전 방식과 비교하여 원하는 전략을 더욱 유연하게 변경할 수 있다.

테스트 코드를 보면 하나의 Context만 생성해두고 

실행 할 때 여러 전략을 인수로 전달하여 실행하는 것을 볼 수 있다.

**실행 그림**

![image](https://user-images.githubusercontent.com/18654358/159111861-76ed80ea-485d-424f-8d1a-5ddaf259902f.png)

1. 클라이언트는 Context를 실행하면서 인수로 Strategy를 전달한다.
2. Context는 execute() 로직을 실행한다.
3. Context는 파라미터로 넘어온 strategy.call() 로직을 실행한다.
4. Context의 execute() 로직이 종료된다.

### 정리

**ContextV1** 은 필드에 Strategy를 지정하는 방식으로 전략 패턴을 구사했다.

- 선 조립, 후 실행의 방법에 적합하다.
    - 스프링의 의존성 주입방식 🙂
- Context를 실행하는 시점에는 **이미 조립이 완료되어 있으므로** 전략을 신경쓰지 않고 
**`단순히 실행만 하면 되는 장점을 가진다.`**

**ContextV2** 는 파라미터에 Strategy를 전달받는 방식으로 전략 패턴을 구사했다.

- 실행할 때마다 전략을 유연히 변경할 수 있다.
- **`단점 역시 실행할 때마다 전략을 지정해줘야 하는 것이다.`**

### 템플릿

지금 우리가 해결하고 싶은 문제는 **“변하는 부분”**과 **“변하지 않는 부분”**을 **`분리`**하는 것이다.

변하지 않는 부분을 템플릿이라 하고

그 템플릿 안에서 변하는 부분에 약간 다른 코드 조각을 넘겨 실행하는 것이 목적이다.

**ContextV1**, **ContextV2** 두가지 방식 다 문제를 해결할 수 있지만

어떤 방식이 더 나아 보이는가?

지금 우리가 원하는 것은 애플리케이션 의존 관계를 설정하는 것 처럼 선 조립, 후 실행이 아니다.

단순히 코드를 실행할 때 변하지 않는 템플릿이 있고,

그 템플릿 안에서 원하는 부분만 살짝 다른 코드를 실행하고 싶을 뿐이다.

***따라서, 우리가 고민하는 문제는 실행 시점에 유연하게 실행 코드 조각을 전달하는 ContextV2가 더 적합하다.***

필드에 다음 전략에 보관하며 사용하는 **ContextV1** 방식의 전략 패턴

인수를 넘겨 전략을 사용하는 **ContextV2** 방식의 전략 패턴

***두가지 전략 패턴에 대해 잘 인지하자. 🙂***

<aside>
💡 디자인 패턴을 공부하면 구조(다이어그램)이 디자인 패턴으로 오해할 수 있다.
물론 디자인 패턴이 맞지만 더욱 중요한 것이 있다.
**`→ 의도`** 이다.

비슷비슷한 디자인 패턴을 구분하는 것은 “의도” 이다.
의도를 구현할 때 ContextV1, ContextV2 와 같이 구현하는 것은 중요치 않다.
구현한 ContextV1, ContextV2가 의도를 달성하는 것인가? 이다.

</aside>
