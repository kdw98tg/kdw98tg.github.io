---
title: "[Design Pattern] 복합 패턴 (Combine Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2024-09-29
last_modified_at: 2024-09-29
---

## 복합패턴으로 Duck 구현하기

2개 이상의 패턴으로 이루어진 패턴을 복합 패턴이라고 합니다.

이번 챕터에서는 복합패턴을 사용하여 DuckSimulator 를 구현합니다. `Adapter`, `Composite`, `Decorator`, `Factory`, `Observer`, `Strategy` 패턴들을 사용하여 구성합니다.

이번 포스팅은 여러가지 패턴을 구현하기 때문에 분량이 많을 것 같습니다.


## Strategy Pattern 을 통해 Quack 할 수 있는 오리들 만들기

일단, 오리들은 전부다 `Quack`이라는 행동을 한다고 가정을 합니다. `Quackable`이란 인터페이스를 만들고, 오리들 클래스들을 만들어서 상속시켜 오리마다의 `Quack` 행동을 정의 합니다.

### Quackable 만들기

```java
package headfirst.designpatterns.combining.ducks;  
  
public interface Quackable {  
    public void quack();  
}
```

### DecoyDuck

```java
package src.combine.ducks;  
  
public class DecoyDuck implements  Quackable{  
    @Override  
    public void quack() {  
        System.out.println("--조용--");  
    }  
}
```

### DuckCall

```java
package src.combine.ducks;  
  
public class DuckCall implements Quackable{  
    @Override  
    public void quack() {  
        System.out.println("--꾸악--");  
    }  
}
```

### MallardDuck

```java
package src.combine.ducks;  
  
public class MallardDuck implements Quackable {  
    @Override  
    public void quack() {  
        System.out.println("--꽥--");  
    }  
}
```

### RedheadDuck

```java
package src.combine.ducks;  
  
import org.w3c.dom.ls.LSOutput;  
  
public class RedheadDuck implements Quackable{  
  
    @Override  
    public void quack() {  
        System.out.println("--꽥--");  
    }  
}
```

### RubberDuck

```java
package src.combine.ducks;  
  
public class RubberDuck implements Quackable{  
    @Override  
    public void quack() {  
        System.out.println("--삑삑--");  
    }  
}
```

### 여기까지 실행 - DuckSimulator

```java
package src.combine;  
  
import src.combine.ducks.*;  
public class DuckSimulator {  
    public static void main(String[] args) {  
        DuckSimulator simulator = new DuckSimulator();  
        simulator.simulate();  
    }  
  
    void simulate() {  
        Quackable mallardDuck = new MallardDuck();  
        Quackable redheadDuck = new RedheadDuck();  
        Quackable duckCall = new DuckCall();  
        Quackable rubberDuck = new RubberDuck();  
  
        System.out.println("\n Duck Simulator");  
  
        simulate(mallardDuck);  
        simulate(mallardDuck);  
        simulate(mallardDuck);  
        simulate(mallardDuck);  
    }  
  
    void simulate(Quackable _quackableDuck) {  
        _quackableDuck.quack();  
    }  
}
```

![](/images/Pasted%20image%2020240929013431.png)

전략 패턴을 적용해서 `Quackable`이라는 공통 인터페이스를 만들고, Duck 들에게 상속시켜 각각의 Duck 마다의 행동을 재정의하여 호출하였습니다.

## Adapter 패턴 적용하기

여기까지 깔끔하게 구현을 했는데, 갑자기 기획자가 와서 오리가 아닌 거위도 만들어 달라고 말을 했습니다.

그래서 저희는 `Goose`라는 클래스를 만들고, `Adapter`를 만들어서 `Quackable` 처럼 동작하도록 만들도록 하겠습니다.

### Goose 만들기

```java
package src.combine.adapter;  
  
public class Goose {  
    public void honk(){  
        System.out.println("--끽끽--");  
    }  
}
```

### GooseAdapter 만들기

Adapter를 만들어서, Goose 도 `Quackable`을 사용할 수 있도록 만들어 줍니다.

```java
package src.combine.adapter;  
  
import src.combine.ducks.Quackable;  
  
public class GooseAdapter implements Quackable {  
    private Goose goose = null;  
  
    public GooseAdapter (Goose _goose){  
        goose = _goose;  
    }  
  
    @Override  
    public void quack() {  
        goose.honk();  
    }  
}
```

### 여기까지 실행해 보기 - DuckSimulator

```java
package src.combine;  
  
import src.combine.adapter.Goose;  
import src.combine.adapter.GooseAdapter;  
import src.combine.ducks.*;  
public class DuckSimulator {  
    public static void main(String[] args) {  
        DuckSimulator simulator = new DuckSimulator();  
        simulator.simulate();  
    }  
  
    void simulate() {  
        Quackable mallardDuck = new MallardDuck();  
        Quackable redheadDuck = new RedheadDuck();  
        Quackable duckCall = new DuckCall();  
        Quackable rubberDuck = new RubberDuck();  
        Quackable goose = new GooseAdapter(new Goose());  
  
        System.out.println("\n Duck Simulator");  
  
        simulate(mallardDuck);  
        simulate(redheadDuck);  
        simulate(duckCall);  
        simulate(rubberDuck);  
        simulate(goose);  
    }  
  
    void simulate(Quackable _quackableDuck) {  
        _quackableDuck.quack();  
    }  
}
```

![](/images/Pasted%20image%2020240929014421.png)

여기까지, Goose 를 Duck 처럼 실행할 수 있도록 만들었습니다.


## 데코레이터로 기능 추가하기

이제 Adpater 까지 끼워서 Goose 를 Duck 처럼 `Quack!` 하도록 만들어 놨더니, 갑자기 오리를 연구하시는 분들이 와서, 오리가 몇번 `Quack`을 했는지 횟수를 세어 달라고 합니다.

이에 저희는 오리 클래스는 수정하지 않고, 기능을 추가하기 위해 `Decorator` 패턴을 사용하여 기능을 추가 합니다.

거위는 이 횟수에 포함시키지 말라고 하네요. 오리만 추가를 해 보겠습니다.

### QuackCounter 구현

타겟 인터페이스를 `Quackable`로 설정한 데코레이터 입니다.
모든 오리가 quack 한 횟수를 세기 위해서 count 는 static 으로 선언 하였습니다.

```java
package src.combine.decorator;  
  
import src.combine.ducks.Quackable;  
  
//타깃 인터페이스 구현  
public class QuackCounter implements Quackable {  
    private Quackable duck = null;  
    private static int numberOfQuacks = 0;//모든 오리가 낸 소리를 세기 위해 static 으로 선언  
  
    public QuackCounter(Quackable _duck){  
        this.duck = _duck;  
    }  
  
    @Override  
    public void quack() {  
        duck.quack();  
        numberOfQuacks++;  
    }  
  
    public static int getQuacksCount(){  
        return numberOfQuacks;  
    }  
}
```

### 여기까지 실행해 보기 - DuckSimulator

```java
package src.combine;  
  
import src.combine.adapter.Goose;  
import src.combine.adapter.GooseAdapter;  
import src.combine.decorator.QuackCounter;  
import src.combine.ducks.*;  
public class DuckSimulator {  
    public static void main(String[] args) {  
        DuckSimulator simulator = new DuckSimulator();  
        simulator.simulate();  
    }  
  
    void simulate() {  
        Quackable mallardDuck = new QuackCounter(new MallardDuck());  
        Quackable redheadDuck = new QuackCounter(new RedheadDuck());  
        Quackable duckCall = new QuackCounter(new DuckCall());  
        Quackable rubberDuck = new QuackCounter(new RubberDuck());  
        Quackable goose = new GooseAdapter(new Goose());//거위는 Counter 에서 제외  
  
        System.out.println("\n Duck Simulator");  
  
        simulate(mallardDuck);  
        simulate(redheadDuck);  
        simulate(duckCall);  
        simulate(rubberDuck);  
        simulate(goose);  
  
        System.out.println("오리가 소리를 낸 횟수 : " + QuackCounter.getQuacksCount() + " 번");  
    }  
  
    void simulate(Quackable _quackableDuck) {  
        _quackableDuck.quack();  
    }  
}
```

![](/images/Pasted%20image%2020240929015454.png)

맨 마지막에 거위를 제외하고, 오리는 4번 울었습니다. 데코레이터를 활용해 성공적으로 기능을 추가하였습니다.

## 추상 팩토리 패턴으로 생성 과정 리펙토링

오리가 quack 한 횟수를 데코레이터 패턴으로 성공적으로 집어넣었지만, 한가지 아쉬운 점이 있습니다. 오리의 `quack` 횟수를 세기 위해서는 오리를 생성할 때, QuackCounter 를 무조건 생성 해야 한다는 것입니다. 사용자가 사용할 때, 이 과정을 뺴먹게 되면, 오리의 `quack`횟수에 오류가 생길 수 있습니다.

그래서 추상 팩토리 패턴으로 이 과정을 리펙토링 합니다.

### 추상 팩토리 클래스 만들기

어떤 객체를 만든다 라는 추상적인 개념만 가진 클래스를 하나 만듭니다.

```java
package src.combine.factory;  
  
import src.combine.ducks.Quackable;  
  
public abstract class AbstractDuckFactory {  
  
    public abstract Quackable createMallardDuck();  
    public abstract Quackable createRedheadDuck();  
    public abstract Quackable createDuckCall();  
    public abstract Quackable createRubberDuck();  
  
}
```

### 추상 팩토리 구현하기

이제 위 추상 클래스를 상속받아, 내용을 정의해 줍니다. Counter 가 달려있는 오리를 생성하는 `CountingDuckFactory`를 만듭니다.

```java
package src.combine.factory;  
  
import src.combine.decorator.QuackCounter;  
import src.combine.ducks.*;  
  
public class CountingDuckFactory extends AbstractDuckFactory{  
    @Override  
    public Quackable createMallardDuck() {  
        return new QuackCounter(new MallardDuck());  
    }  
  
    @Override  
    public Quackable createRedheadDuck() {  
        return new QuackCounter(new RedheadDuck());  
    }  
  
    @Override  
    public Quackable createDuckCall() {  
        return new QuackCounter(new DuckCall());  
    }  
  
    @Override  
    public Quackable createRubberDuck() {  
        return new QuackCounter(new RubberDuck());  
    }  
}
```

### 여기까지 실행해 보기 - DuckSimulator

이제, 추상 클래스를 만들어, 그 추상클래스의 `create...` 메소드를 사용하여 오리를 생성 합니다.
물론 Factory에서 데코레이터를 달아놨으니, 카운터를 빼먹을 실수는 할 수 없습니다.

```java
package src.combine;  
  
import src.combine.adapter.Goose;  
import src.combine.adapter.GooseAdapter;  
import src.combine.decorator.QuackCounter;  
import src.combine.ducks.*;  
import src.combine.factory.AbstractDuckFactory;  
import src.combine.factory.CountingDuckFactory;  
  
public class DuckSimulator {  
    public static void main(String[] args) {  
        DuckSimulator simulator = new DuckSimulator();  
        AbstractDuckFactory factory = new CountingDuckFactory();  
        simulator.simulate(factory);  
    }  
  
    void simulate(AbstractDuckFactory _countingDuckFactory) {  
        Quackable mallardDuck = _countingDuckFactory.createMallardDuck();  
        Quackable redheadDuck = _countingDuckFactory.createRedheadDuck();  
        Quackable duckCall = _countingDuckFactory.createDuckCall();  
        Quackable rubberDuck = _countingDuckFactory.createRubberDuck();  
        Quackable goose = new GooseAdapter(new Goose());//거위는 Counter 에서 제외  
  
        System.out.println("\n Duck Simulator");  
  
        simulate(mallardDuck);  
        simulate(redheadDuck);  
        simulate(duckCall);  
        simulate(rubberDuck);  
        simulate(goose);  
  
        System.out.println("오리가 소리를 낸 횟수 : " + QuackCounter.getQuacksCount() + " 번");  
    }  
  
    void simulate(Quackable _quackableDuck) {  
        _quackableDuck.quack();  
    }  
}
```

![](/images/Pasted%20image%2020240929020530.png)

같은 결과가 반환 됩니다.

## 여러 오리를 컴포지트 패턴을 사용해 한번에 관리하기

이번엔 다른 개발자가 와서 제 코드를 보더니 이런 말을 합니다.

와.. 오리 종류 진짜 많네요... 이 오리들 전부 같은 역할을 하는 것 같은데 하나로 관리를 못하는건가요? 저였으면 하나로 관리 할 것 같네요.

이 말을 듣고 긁힌 저는 이 오리들을 한번에 관리하기 위해서 컴포지트 패턴을 사용하여 오리들을 관리 하도록 설계를 합니다.


### 오리 무리들을 관리할 FLock 클래스 생성

타겟 인터페이스를 `Quackable`로 잡은 후, 리스트를 만들어서 `Quackable`오브젝트를 담을 수 있도록 만듭니다.

그리고 `Iterator` 패턴을 사용하여, `quack` 메소드 호출 시, 리스트를 순회하며 리스트에 들어있는 `Quackable`의 `quack`메소드를 호출하도록 합니다.

```java
package src.combine.composite;  
  
import src.combine.ducks.Quackable;  
  
import java.util.ArrayList;  
import java.util.Iterator;  
import java.util.List;  
  
//오리 무리 클래스  
public class Flock implements Quackable {  
  
    List<Quackable> quackers = new ArrayList<Quackable>();  
  
    public void add(Quackable _duck) {  
        quackers.add(_duck);  
    }  
  
    @Override  
    public void quack() {  
        Iterator<Quackable> iterator = quackers.iterator();  
  
        while (iterator.hasNext()) {  
            Quackable quackable = iterator.next();  
            quackable.quack();  
        }  
    }  
}
```

### 여기까지 실행해 보기 - DuckSimulator

이제, 오리들을 각각 quack 시키지 말고, Flock 이라는 복합체에 넣어서 한번의 `quack`메소드로 전부 `Quack!`하게 수정해 봅시다.

```java
package src.combine;  
  
import src.combine.adapter.Goose;  
import src.combine.adapter.GooseAdapter;  
import src.combine.composite.Flock;  
import src.combine.decorator.QuackCounter;  
import src.combine.ducks.*;  
import src.combine.factory.AbstractDuckFactory;  
import src.combine.factory.CountingDuckFactory;  
  
public class DuckSimulator {  
    public static void main(String[] args) {  
        DuckSimulator simulator = new DuckSimulator();  
        AbstractDuckFactory factory = new CountingDuckFactory();  
        simulator.simulate(factory);  
    }  
  
    void simulate(AbstractDuckFactory _countingDuckFactory) {  
  
        Quackable mallardDuck = _countingDuckFactory.createMallardDuck();  
        Quackable redheadDuck = _countingDuckFactory.createRedheadDuck();  
        Quackable duckCall = _countingDuckFactory.createDuckCall();  
        Quackable rubberDuck = _countingDuckFactory.createRubberDuck();  
        Quackable goose = new GooseAdapter(new Goose());//거위는 Counter 에서 제외  
  
        System.out.println("\n Duck Simulator");  
  
        //전체 오리들 관리하는 컴포지트  
        Flock flocks = new Flock();  
        flocks.add(redheadDuck);  
        flocks.add(duckCall);  
        flocks.add(rubberDuck);  
        flocks.add(goose);  
  
  
        //진짜 오리들 관리하는 컴포지트  
        Flock realDuck = new Flock();  
  
        Quackable mallardDuck1 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck2 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck3 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck4 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck5 =  _countingDuckFactory.createMallardDuck();  
  
        realDuck.add(mallardDuck);  
        realDuck.add(mallardDuck1);  
        realDuck.add(mallardDuck2);  
        realDuck.add(mallardDuck3);  
        realDuck.add(mallardDuck4);  
        realDuck.add(mallardDuck5);  
  
        flocks.add(realDuck);//전체 오리 리스트에 진짜 오리 리스트 추가  
  
  
        System.out.println("\n 전체 오리들 ");  
        simulate(flocks);  
  
        System.out.println("\n 진짜 오리들 ");  
        simulate(realDuck);  
  
  
        System.out.println("오리가 소리를 낸 횟수 : " + QuackCounter.getQuacksCount() + " 번");  
    }  
  
    void simulate(Quackable _quackableDuck) {  
        _quackableDuck.quack();  
    }  
}
```


![](/images/Pasted%20image%2020240929022135.png)

CountingDuckFactory 로 만든 객체로 이루어져서, 거위가 꽥 한거 빼고 15번 모두 정확하게 세어졌네요.

## 누가 소리를 꽥 소리를 내었는가 - 옵저버 패턴

이제 마지막으로 꽥꽥학자가 난입하여 누가 소리를 내었는가를 확인한다고 합니다.

이 기능을 옵저버 패턴으로 구현해 봅니다.

### QuackObservable 인터페이스 및 Observer 인터페이스 만들기

```java
package src.combine.observer;  
  
public interface QuackObservable {  
    public void registerObserver(Observer _observer);  
}
```

```java
package src.combine.observer;  
  
public interface Observer {  
    public void update(QuackObservable _duck);  
}
```

### Quackable 에 QuackObserver 인터페이스 상속

이제 Quackable 기능이 있는 친구들은 추적 가능해야 하기 때문에 `Quackable`에 `QuackObserver`를 상속 시켜 줍니다.

```java
package src.combine.ducks;  
  
import src.combine.observer.QuackObservable;  
  
public interface Quackable extends QuackObservable {  
    public void quack();  
  
}
```

이렇게 되면 `Quackable` 을 상속 받는 친구들은 모두 `registerObserver` 라는 함수를 구현해주어야 합니다. 미친듯이 빨간줄이 뜨겠군요



### Observable 클래스 만들기

`Observable` 클래스를 만들어서, `Observer` 인터페이스를 상속받는 친구들을 모두 리스트에 담아, 그 리스트들을 `Iterator`가 순회하도록 만듭니다.

```java
package src.combine.observer;  
  
import java.util.ArrayList;  
import java.util.Iterator;  
import java.util.List;  
  
public class Observable implements QuackObservable {  
  
    private List<Observer> observerList = new ArrayList<Observer>();  
    private QuackObservable duck = null;  
  
    public Observable(QuackObservable _duck) {  
        this.duck = _duck;  
    }  
  
    @Override  
    public void registerObserver(Observer _observer) {  
        observerList.add(_observer);  
    }  
  
    public void notifyObservers(){  
        Iterator<Observer> iterator = observerList.iterator();  
        while(iterator.hasNext()){  
            Observer observer = iterator.next();  
            observer.update(duck);  
        }  
    }  
  
}
```

이제 Quackable 을 상속받는 친구들을 고쳐 봅시다. 

편의상 `MallardDuck`만 포스팅 하겠습니다.

```java
package src.combine.ducks;  
  
import src.combine.observer.Observable;  
import src.combine.observer.Observer;  
  
  
public class MallardDuck implements Quackable {  
    private Observable observable = null;  
  
    public MallardDuck() {  
        observable = new Observable(this);  
    }  
    @Override  
    public void quack() {  
        System.out.println("--꽥--");  
        notifyObservers();  
    }  
  
    @Override  
    public void registerObserver(Observer _observer) {  
        observable.registerObserver(_observer);  
    }  
  
    public void notifyObservers(){  
        observable.notifyObservers();  
    }  
}
```

### 꽥꽥학자 만들기

이제 옵저버들이 준비가 되었으니, 꽥꽥 학자 클래스를 만들어서 오리들을 추적해 봅시다.

```java
package src.combine.observer;  
  
public class Quackologist implements Observer{  
    @Override  
    public void update(QuackObservable _duck) {  
        System.out.println("꽥꽥 학자: "+ _duck + "가 방금 소리를 냄");  
    }  
}
```

### 완성된 DuckSimulator

이제 모든 기능이 완성되었습니다. 오리들을 `Iterator`패턴으로 순회시키고, `Strategy` 패턴으로 각기 다른 꽥 소리를 내며, `Abstract Factory`와 `Decorator` 패턴을 사용해 카운팅 기능을 만들었고, 마지막으로 `Composite` 와 `Observer`를 사용하여 오리의 울음소리 횟수와 어떤 오리가 소리를 내었는지 추적도 가능해 졌습니다.

```java
package src.combine;  
  
import src.combine.adapter.Goose;  
import src.combine.adapter.GooseAdapter;  
import src.combine.composite.Flock;  
import src.combine.decorator.QuackCounter;  
import src.combine.ducks.*;  
import src.combine.factory.AbstractDuckFactory;  
import src.combine.factory.CountingDuckFactory;  
import src.combine.observer.Quackologist;  
  
public class DuckSimulator {  
    public static void main(String[] args) {  
        DuckSimulator simulator = new DuckSimulator();  
        AbstractDuckFactory factory = new CountingDuckFactory();  
        simulator.simulate(factory);  
    }  
  
    void simulate(AbstractDuckFactory _countingDuckFactory) {  
  
        Quackable mallardDuck = _countingDuckFactory.createMallardDuck();  
        Quackable redheadDuck = _countingDuckFactory.createRedheadDuck();  
        Quackable duckCall = _countingDuckFactory.createDuckCall();  
        Quackable rubberDuck = _countingDuckFactory.createRubberDuck();  
        Quackable goose = new GooseAdapter(new Goose());//거위는 Counter 에서 제외  
  
        System.out.println("\n Duck Simulator");  
  
        //전체 오리들 관리하는 컴포지트  
        Flock flocks = new Flock();  
        flocks.add(redheadDuck);  
        flocks.add(duckCall);  
        flocks.add(rubberDuck);  
        flocks.add(goose);  
  
  
        //진짜 오리들 관리하는 컴포지트  
        Flock realDuck = new Flock();  
  
        Quackable mallardDuck1 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck2 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck3 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck4 =  _countingDuckFactory.createMallardDuck();  
        Quackable mallardDuck5 =  _countingDuckFactory.createMallardDuck();  
  
        realDuck.add(mallardDuck);  
        realDuck.add(mallardDuck1);  
        realDuck.add(mallardDuck2);  
        realDuck.add(mallardDuck3);  
        realDuck.add(mallardDuck4);  
        realDuck.add(mallardDuck5);  
  
        flocks.add(realDuck);//전체 오리 리스트에 진짜 오리 리스트 추가  
  
  
        Quackologist quackologist = new Quackologist();  
        flocks.registerObserver(quackologist);  
  
        simulate(flocks);  
  
        System.out.println("오리가 소리를 낸 횟수 : " + QuackCounter.getQuacksCount() + " 번");  
  
    }  
  
    void simulate(Quackable _quackableDuck) {  
        _quackableDuck.quack();  
    }  
}
```

![](/images/Pasted%20image%2020240929024941.png)

## 정리

여기까지 여러가지 패턴을 사용해서 오리 울음소리 기능을 만들어 보았습니다. 하나의 기능을 만드는데도 여러가지 패턴이 쓰이는군요 .... 

그래도 여러가지 패턴을 기능의 추가에 맞춰서 구현해 보니, 그냥 하나의 패턴만 구현할 때 보다 훨씬 더 이해가 잘 되었습니다.

책을 안보고도 이렇게 설계할 수 있는 날을 기대하며, 더 열심히 공부해야 겠습니다.