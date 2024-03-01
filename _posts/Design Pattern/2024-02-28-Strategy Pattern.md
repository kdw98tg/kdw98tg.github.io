---
title: "[Design Pattern] 전략 패턴 (Strategy Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2023-05-12
last_modified_at: 2024-03-01
---



## 전략 패턴이란?

> 전략패턴 (Strategy Pattern)은 알고리즘군을 정의하고 캡슐화해서 각각의 알고리즘군을 수정해서 쓸 수 있게 할 수 있는 패턴 입니다. 
> 
> 전략 패턴을 사용하면 클라이언트로부터 알고리즘을 분리해서 독립적으로 변경할 수 있습니다.

즉, 전략패턴은 알고리즘의 변화에 따라 동적으로 알고리즘을 선택할 수 있도록 해주며, 유연성과 확장성을 제공합니다.

### 전략 패턴이 사용되는 곳
- 동일한 문제를 해결하는 여러 알고리즘이 존재할 때
- 알고리즘을 사용하는 클라이언트와 알고리즘을 분리하고자 할 때
- 실행 중에 알고리즘을 교체할 때

### 전략패턴의 이점
- `유연성` : 알고리즘을 필요에 따라 교체하기 쉬움
- `확장성` : 새로운 전략을 추가하기 쉬움
- `코드 재사용` : 필요한 알고리즘을 재사용할 수 있음

전략패턴은 다음과 같은 디자인 원칙을 따릅니다.

### 디자인 원칙
> 애플리케이션에서 달라지는 부분을 찾아내고, 달라지지 않는 곳과 분리한다.

**즉, 바뀌는 부분은 따로 뽑아서 캡슐화 하는 것입니다. 그렇다면 나중에 바뀌지 않는 부분에는 영향을 주지 않고, 바뀌어야 하는 부분을 고치거나 확장을 하기 쉽습니다.**

## 예제

여러가지의 오리를 구현해봅시다.

오리에는 여러가지 종류가 있습니다.

`진짜 오리`, `나무오리`, `러버덕`, `날지 못하는 오리`, `시기에 따라 사는곳을 옮기는 오리` 등등 여러가지 오리가 있습니다.

이것들을 막연하게 구현하려면, 각 클래스마다 오리의 행동을 정의해야 합니다.

하지만, 이렇게 하면 모든 구상클래스에서 오리의 행동을 정의해야 합니다.

지금은 5개정도의 오리를 구현하기 때문에 각기 클래스마다 정의해서 사용해도 되지만, 이 오리 클래스가 100개 ... 10000개 가 되어버리면, 유지보수가 불가능한 수준이 되어 버립니다.

그렇기 때문에 전략패턴을 구현해서, 해당 로직을 구현합니다.

이때, 핵심은 `객체지향의 다형성` 을 이용하여 전략 패턴을 구현합니다. 즉, 실제 실행 시에 쓰이는 객체가 코드에 고정되지 않도록 상위 형식에 맞춰 프로그래밍해서 다형성을 활용해야 한다는 것입니다.

각각 다른 오리의 `나는 행동(fly behavior)`, `우는 행동(quack behavior)` 를 구현해 봅시다.

### 행동에 따른 인터페이스 정의

우선 각각의 행동에 따른 인터페이스를 생성해 줍니다.

🗅 **interface FlyBehavior.java**
```java
package src.strategy.behavior.interfaces;  
  
//오리가 날아다니는 행동을 정의할 인터페이스  
public interface FlyBehavior {  
    public void fly();  
}
```

🗅 **interface QuackBehavior**
```java
package src.strategy.behavior.interfaces;  
  
//오리가 우는 소리를 정의할 인터페이스  
public interface QuackBehavior {  
    public void quack();  
}
```


위와같이 정의하면, 각기 다른 Duck이 호출하여, 자신만의 `fly()`, `quack()` 메소드를 재정의 해주기만 하면 됩니다.



### 인터페이스를 상속받아 구현할 구상 클래스 정의

이제 인터페이스를 상속받아 실제 행동을 구현한 클래스를 만듭니다.

간단하게 `fly with wings`, `quack` 두가지 행동을 구현해 봅니다.

🗅 **class FlyWithWings**
```java
package src.strategy.behavior;  
  
import src.strategy.behavior.interfaces.FlyBehavior;  
  
public class FlyWithWings implements FlyBehavior {  
    @Override  
    public void fly() {  
        System.out.println("날고 있어요!");  
    }  
}
```

🗅 **class Quack**
```java
package src.strategy.behavior;  
  
import src.strategy.behavior.interfaces.QuackBehavior;  
  
public class Quack implements QuackBehavior {  
    @Override  
    public void quack() {  
        System.out.println("꽥");  
    }  
}
```

### 오리의 추상 클래스 제작

이제 이 행동들을 할 오리를 정의해 봅니다.

오리 역시 여러가지 형태로 존재할 수 있기에 추상 클래스로 구현을 합니다.

어떤 오리를 구현하든 Duck을 상속받아 구현합니다.

그때에, 앞서 정의한 행동들을 자식 클래스에 주입시켜, 행동하도록 합니다.

🗅 **class Duck**
```java
package src.strategy.duck;  
  
import src.strategy.behavior.interfaces.FlyBehavior;  
import src.strategy.behavior.interfaces.QuackBehavior;  
  
public abstract class Duck {  
  
    //행동의 인터페이스 구현  
    FlyBehavior flyBehavior = null;  
    QuackBehavior quackBehavior = null;  
  
    public void setFlyBehavior(FlyBehavior _flyBehavior){  
        flyBehavior = _flyBehavior;  
    }  
    public void setQuackBehavior(QuackBehavior _quackBehavior){  
        quackBehavior = _quackBehavior;  
    }  
  
    //오리의 행동을 출력할 추상함수  
    public abstract void display();  
  
    //나는 행동 실행부  
    public void performFly(){  
        flyBehavior.fly();  
    }  
      
    //우는 행동 실행부  
    public void performQuack(){  
        quackBehavior.quack();  
    }  
      
    //재구현할 필요 없는 메서드  
    public void swim(){  
        System.out.println("모든 오리는 물에 뜸");  
    }  
}
```

### 실제 인스턴스로 사용할 오리 제작

이제 실제로 인스턴스화 시킬 오리 클래스를 만들어 봅니다.

오리는 `MallardDuck`, `ModelDuck` 두종류를 만들어 보겠습니다.

부모 클래스에서 정의한 인터페이스 변수에 `다형성`을 이용하여 각자의 클래스에 구현해야 할 행동을 정의한 클래스를 생성 합니다.


🗅 **class MallardDuck**
```java
public class MallardDuck extends Duck{  
  
    public MallardDuck(){  
        quackBehavior = new Quack();  
        flyBehavior = new FlyWithWings();  
    }  
  
    @Override  
    public void display() {  
        System.out.println("저는 물오리 입니다.");  
    }  
}
```

🗅 **class ModelDuck**
```java
public class MallardDuck extends Duck{  
  
    public MallardDuck(){  
        quackBehavior = new Quack();  
        flyBehavior = new FlyWithWings();  
    }  
  
    @Override  
    public void display() {  
        System.out.println("저는 물오리 입니다.");  
    }  
}
```

### Duck 실행

이제 각자 행동을 구현한 후 Main 함수에서 구동시켜 봅니다.

Main 함수에서도 `다형성`을 활용하여 Duck을 생성하고, 행동해야할 함수를 호출 합니다.

🗅 **class ModelDuck**
```java
public class Main {  
    public static void main(String[] args) {  
        Duck mallard = new MallardDuck();  
        mallard.display();  
        mallard.performQuack();  
        mallard.performFly();  
  
        System.out.println("\n");  
  
        Duck modelDuck  = new ModelDuck();  
        modelDuck.display();  
        //생성자로 넣으면 행동을 동적으로 정의할 수 있음  
        modelDuck.setFlyBehavior(new FlyRocketPowered());  
        modelDuck.performQuack();  
        modelDuck.performFly();  
    }  
}
```

![](/images/Pasted%20image%2020240301201656.png)

다음과 같이 각자 정의한 클래스를 호출하여 다른 오리 객체가 생성된 것을 확인할 수 있습니다.
