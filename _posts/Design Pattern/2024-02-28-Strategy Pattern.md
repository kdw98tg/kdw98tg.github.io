---
sidebar:
  nav: docs
author_profile: false
toc: true
tags: [python, blog, jekyll]
categories: 코딩
title: "[디자인 패턴] Strategy Pattern"
layout: single
---

![Strategy Pattern](/images/Pasted%20image%2020240228200824.png)

전략 패턴(Strategy Pattern)은 객체지향 디자인 패턴 중 하나로, 동일한 문제를 해결하기 위한 여러 알고리즘(전략)을 정의하고 각각을 캡슐화하여 교환 가능하게 만드는 패턴 이다.

​

또한 알고리즘군을 정의하고 캡슐화해서 각각의 알고리즘군을 수정해서 쓸 수 있게 해준다. 전략패턴을 사용하면 클라이언트로부터 알고리즘을 분리해서 독립적으로 변경할 수 있다.

​

이는 알고리즘의 변화에 따라 동적으로 알고리즘을 선택할 수 있도록 해주며, 유연성과 확장성을 제공한다.

​

전략 패턴은 주로 다음과 같은 상황에서 사용된다.

- 동일한 문제를 해결하는 여러 알고리즘이 존재할때.
    
- 알고리즘을 사용하는 클라이언트와 알고리즘음을 분리하고자 할때.
    
- 실행 중에 알고리즘을 교체해야 할 때.
    

​

전략 패턴의 주요 이점은 다음과 같다.

- 유연성 : 전략 패턴은 알고리즘을 쉽게 교체할 수 있도록 해준다. 클라이언트는 알고리즘의 구체적인 세부 사항을 알 필요 없이 전략 인터페이스와 상호작용 할 수 있다.
    


- 확장성 : 새로운 전략을 추가하기가 쉽다. 전략 인터페이스를 구현하는 클래스를 작성하고, 컨텍스트에서 해당 전략을 사용하도록 설정하기만 하면 된다.
    

​

- 코드 재사용 : 전략 패턴은 알고리즘을 공통으로 사용할 수 있도록 해준다. 각각의 전략은 독립적으로 재사용될 수 있다. 다른 전략과 조합하여 사용할 수 있다.
    

​

간단한 예시를 들어본다.

​

오리를 만들려고 한다. Duck 이라는 추상클래스를 만들고 거기에 quack() 메소드와 fly()메소드를 만든다고 생각한다. 일반적인 Duck 이라면 모두 quack()하고 fly() 하지만 예외가 있다.

​

만약 고무오리를 만든다고 생각해보자. 고무오리는 quack()하지도 않고 fly() 하지도 않는다. 이때 그러면 super 클래스에 고무오리만을 위한 메서드를 만들어야 한다.

​

이런식으로 모든 경우의 수를 대비할 수 없기 때문에 인터페이스를 만들어 유동적으로 바꿔끼며 사용한다.

​

우선 인터페이스를 만든다.

​

quack()과 fly()만을 생각하고 만든다.

​

public interface FlyBehaviour { public void fly(); }

public interface QuackBehaviour { public void quack(); }

이렇게 두가지의 인터페이스를 만든다.

​

그리고 이 인터페이스를 상속받은 각각 행동을 정의하는 클래스를 만든다. 그리고 그 함수를 정의한다.

​

public class FlyNoWay implements FlyBehaviour { @Override public void fly() { System.out.println("저는 못날아요"); } }

public class FlyWithWings implements FlyBehaviour { @Override public void fly() { System.out.println("날고 있어요!!"); } }

public class Quack implements QuackBehaviour { @Override public void quack() { System.out.println("꽥"); } }

public class Squeak implements QuackBehaviour { @Override public void quack() { System.out.println("삑"); } }

이런식으로 정의했다.

​

그럼 이제 Duck 이라는 추상 클래스를 만들어 본다.

​

public abstract class Duck { FlyBehaviour flyBehaviour; QuackBehaviour quackBehaviour; public Duck(){}; //생성자 public abstract void display(); public void performFly() { flyBehaviour.fly(); } public void performQuack() { quackBehaviour.quack(); } public void swim() { System.out.println("모든 오리는 물에 뜹니다. 가짜 오리도 뜨죠"); } }

이 Duck 이라는 추상클래스는 FlyBehaviour 와 QuackBehaviour 라는 인터페이스를 가지고 있다.

​

그리고 무슨 행동이 들어올지는 모르겠지만 그 행동들의 상위 인터페이스인

performFly() 함수에는 flybehaviour.fly()

performQuack() 함수에는 quackBehaviour.quack() 함수를 정의한다.

​

그리고 이 추상 클래스를 상속받는 MallardDuck 클래스를 만든다.

​

import java.rmi.MarshalException; public class MallardDuck extends Duck { public MallardDuck() { quackBehaviour = new Quack(); flyBehaviour = new FlyWithWings(); } @Override public void display() { System.out.println("저는 물오리 입니다."); } }

이 오리는 생성될때 quackbehaviour 는 Quack() 에서 정의한 메서드, flyBehaivour에서 정의한 FlyWithWings() 메서드를 정의한다.

​

그리고 실행부를 정의한다.

​

다형성을 위해 Duck 으로 생성을 한다.

// Press Shift twice to open the Search Everywhere dialog and type `show whitespaces`, // then press Enter. You can now see whitespace characters in your code. public class Main { public static void main(String[] args) { Duck mallard = new MallardDuck(); mallard.performQuack(); mallard.performFly(); } }

이렇게 설정을 하면 슈퍼클래스에서 모든 행동을 정의하지 않았지만 각 상황에 맞는 메서드를 정의할 수 있다.

​

![](https://postfiles.pstatic.net/MjAyMzA1MTJfMTYy/MDAxNjgzODk5ODIyOTIy.FueSGTyepCmXV6CAPgoFoyolzshwRiwM_rFz2b_8n7Ig.m_7TD-vBRmpt22XHr6SQNlQQ_ol02CcpGe9GTrIH_tQg.PNG.98tg/image.png?type=w773)

​

​

이제 전략패턴의 꽃인 동적으로 알고리즘 변경을 해보자.

​

flyBehaviour 와 quackBehaviour를 세팅해주는 세터를 설정한다.

​
```java
public abstract class Duck  
{  
    FlyBehaviour flyBehaviour;  
    QuackBehaviour quackBehaviour;  
  
    public Duck(){}; //생성자  
  
    public abstract void display();  
    public void performFly()  
    {  
       flyBehaviour.fly();  
    }  
    public void performQuack()  
    {  
       quackBehaviour.quack();  
    }  
    public void swim()  
    {  
       System.out.println("모든 오리는 물에 뜹니다. 가짜 오리도 뜨죠");  
    }  
    public void setFlyBehavior(FlyBehaviour _fb)  
    {  
       flyBehaviour = _fb;  
    }  
    public void setQuackBehaviour(QuackBehaviour _qb)  
    {  
       quackBehaviour = _qb;  
    }  
}
```


그리고 ModelDuck을 만든다.

​

public class ModelDuck extends Duck { public ModelDuck() { flyBehaviour = new FlyNoWay(); quackBehaviour = new Quack(); } @Override public void display() { System.out.println("저는 모형 오리 입니다"); } }

그리고 로켓으로 날아가는 클래스를 하나 만든다,

​

public class FlyRocketPowered implements FlyBehaviour { @Override public void fly() { System.out.println("로켓 추진으로 날아갑니다."); } }

메인함수에서 동적으로 코드를 바꿔본다.

// Press Shift twice to open the Search Everywhere dialog and type `show whitespaces`, // then press Enter. You can now see whitespace characters in your code. public class Main { public static void main(String[] args) { Duck mallard = new MallardDuck(); mallard.performQuack(); mallard.performFly(); Duck model = new ModelDuck(); model.performFly(); model.setFlyBehavior(new FlyRocketPowered()); model.performFly(); } }

런타임도중에 동적으로 알고리즘이 바뀐것을 볼 수 있다.

​

![](https://postfiles.pstatic.net/MjAyMzA1MTRfNzkg/MDAxNjg0MDY3MzgzMTAx.wd5oXyFRsT5KvjxxyilWeuh_tLIIeggypOKfEfLOmJ4g._IOuZfgoqeQKDHmnXstBFRy2KsB7pD6GMcYLaQS60Kwg.PNG.98tg/image.png?type=w773)