---
title: "[Design Pattern] 데코레이터 패턴 (Decorator Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2023-03-02
last_modified_at: 2024-03-02
---

## 데코레이터 패턴이란?

>데코레이터 패턴(Decorator Pattern)으로 객체에 추가 요소를 동적으로 더할 수 있습니다.
>
>데코레이터를 사용하면 서브클래스를 만들 때보다 훨씬 유연하게 기능을 확장할 수 있습니다.

### 데코레이터 패턴이 사용되는 곳
- 클래스의 요소들을 계속해서 수정을 하면서 사용하는 구조
- 여러 요소들을 조합해서 사용하는 클래스 구조

### 데코레이터 패턴의 장점
- 기존 코드를 수정하지 않고도 데코레이터 패턴을 통해 행동을 확장 가능
- 구성와 위임을 통해서 실행중에 새로운 행동을 추가

### 데코레이터 패턴의 단점
- 의미없는 객체들이 너무 많이 추가될 수 있음
- 과사용시 코드가 필요 이상으로 복잡해질 수 있음

### 사용된 디자인 원칙
> OCP (Open-Closed Principle) 이 적용
> 
> 클래스는 확장에는 열려 있어야 하지만, 변경에는 닫혀 있어야 한다.

즉, 디자인한 것중에서 가장 바뀔 가능성이 높은 부분을 중점적으로 살펴보고, OCP를 적용하는 방법이 가장 좋습니다.

반면, 무조건적으로 OCP를 적용한다면, 괜히 쓸데없는 일을 하며 시간을 낭비할 수 있으며, 필요 이상으로 복잡하고 이해하기 힘든 코드를 만들게 되는 부작용이 발생할 수 있습니다.

**클래스를 수정하지 않고, 필요한 클래스를 동적으로 추가할 수 있다는 점**에서 데코레이터 패턴은 `OCP`를 준수하고 있다고 할 수 있습니다.


## 예제

스타버즈(starbuzz)라는 가상의 커피집을 구성해 봅시다.

커피에는 여러가지 종류가 존재합니다.

예를들어, 아메리카노를 시키는 상황을 생각해 봅시다.

기본의 아메리카노를 시키는 경우도 있지만, 사람에 따라 휘핑크림을 추가한다던지, 샷을 추가한다던지, 시럽을 1펌프 넣는다던지 여러가지 상황이 생길 수 있습니다.

해당사항들을 모두 구상클래스에서 정의한다는 것은 아주 힘든 일입니다.... 휘핑 크림을 100번, 샷을 203번 추가하는 고객이 나타날 수 있기 때문이죠.

그렇기 때문에 기본 메뉴 (아메리카노, 카페라떼, 허브티 ...)을 정의 한 다음, 주문에 맞게 동적으로 추가 사항들을 추가하는 것이 합리적입니다.

이것들을 가능하게 해주는 패턴이 `데코레이터(Decorator)` 패턴 입니다.

### Beverage 구현하기

우선, 여러가지의 `Beverage`들이 있기 때문에, 해당 클래스를 아우를 `Beverage` 라는 추상 클래스를 만듭니다.

이 추상 클래스는 이 커피가 어떤 커피인지 알려주는 `getDescriptions()` 와 size를 설정하는 `setSize()`부분, 그리고 이 클래스를 상속받는 Beverages 가 재구현할 `cost()` 함수가 있습니다.

🗅 **abstract class Beverage**
```java
public abstract class Beverage {  

    public enum Size{TALL, GRANDE, VENTI};  
    
    Size size = Size.TALL;  
    
    String description = "Unknown Beverage";  
    
    public String getDescription(){  
        return description;  
    }  
    
    public void setSize(Size size){  
        this.size = size;  
    }  
    
    public Size getSize(){return this.size;}  
    
    public abstract double cost();  
}
```

위 메서드를 상속받아 각 Beverage를 구현해 봅니다.

간단하게 `DarkRoast` 커피와 `HouseBlend` 커피를 구현해 봅시다.


🗅 **class DarkRoast**
```java
public class DarkRoast extends Beverage{  
  
    public DarkRoast(){  
        description = "다크 로스팅된 커피";  
    }  
  
    @Override  
    public double cost() {  
        return 3.5;  
    }  
}
```

🗅 **class HouseBlend**
```java
public class HouseBlend extends Beverage{  
  
    public HouseBlend(){  
        description = "하우스 블렌드 커피";  
    }  
    @Override  
    public double cost() {  
        return .89;  
    }  
}
```


### Beverage 첨가물 구현하기

이제 기본 음료들을 구현했으니, 음료에 들어갈 첨가물들을 구현해 봅니다.

`Mocha` 와 `Whip` 총 2가지를 구현해 보겠습니다.

우선 첨가물들도 여러가지가 생길 수 있기 때문에, 추상클래스를 만들어 이들을 묶어 줍니다.

이때, 주의해야 할 점은 **첨가물 클래스는 음료 클래스를 상속받게 된다는 것입니다.**

이는 생성자에서 `Beverage`를 받기 위함 입니다.

생성자에서 `Beverage`를 받아 그 `Beverage`에 추가 로직을 구현해, 장식하는 것이죠.

🗅 **abstract class CondimentDecorator**
```java
public abstract class CondimentDecorator extends Beverage//Beverage가 들어갈 자리에 들어갈 수 있도록 상속받음  
{  
    //각 데코레이터가 감쌀 음료를 나타내는 Beverage 객체를 여기에 지정  
    Beverage beverage;  
      
    //모든 첨가물에서 getDescription 메소드를 재구현  
    public abstract String getDescription();  
    public Size getSize(){return beverage.getSize();}  
}
```

그 후, `CondimentDecorator`를 상속받아 구상하는 클래스를 만들도록 합니다.

구현클래스에는 `cost()`라는 함수를 통해 첨가물만큼의 가격을 결정하도록 합니다.

🗅 **class Mocha**
```java
//데코레이터  
public class Mocha extends CondimentDecorator {  
  
    public Mocha(Beverage beverage) {  
        this.beverage = beverage;  
    }  
  
    @Override  
    public double cost() {  
        double cost = beverage.cost();  
        if (beverage.getSize() == Size.TALL) {  
            cost += .20;  
        } else if (beverage.getSize() == Size.GRANDE) {  
            cost += .30;  
        } else if (beverage.getSize() == Size.VENTI) {  
            cost += .40;  
        }  
        return cost;  
    }  
  
    @Override  
    public String getDescription() {  
        return beverage.getDescription() + ", 모카";  
    }  
}
```

🗅 **class Mocha**
```java
public class Whip extends CondimentDecorator{  
  
    public Whip (Beverage beverage){  
        this.beverage = beverage;  
    }  
    @Override  
    public double cost() {  
        return beverage.cost() + .50;
    }  
  
    @Override  
    public String getDescription() {  
        return beverage.getDescription() + ", 휘핑크림";  
    }  
}
```

### Main 함수 실행

이제 `Beverage`와 `CondimentDecorator` 들이 완성되었으니, Main 클래스에서 작동을 해보도록 합니다.

저는 `DarkRoat`커피에 `Whip` 2개와, `Mocha` 1개를 첨가해 보겠습니다.

우선, `Beverage`를 생성한 후, 넣고싶은 첨가물을 생성하고, 생성자로 `Beverage`를 넣어주는 방식 입니다.

🗅 **class Mocha**
```java
public class StarbuzzCoffee {  
  
    public static void main(String[] args) {   
  
        Beverage beverage2 = new DarkRoast();  
        beverage2 = new Mocha(beverage2);  
        beverage2 = new Mocha(beverage2);  
        beverage2 = new Whip(beverage2);  
        System.out.println(beverage2.getDescription()+" $"+String.format("%.2f",beverage2.cost()));  
    }  
  
}
}
```

![실행결과](/images/Pasted%20image%2020240303165702.png)

위와같이 첨가한 모든 첨가물과 그에 해당하는 가격이 출력되는것을 볼 수 있습니다.

여담이지만, `Java`의 `java.io` 패키지는 데코레이터 패턴으로 만들어져 있다고 합니다.