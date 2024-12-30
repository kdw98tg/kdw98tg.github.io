---
title: "[Winform] UI 커스터마이징"

categories:
  -  Winform
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2024-12-30
last_modified_at: 2024-12-30
---

## 플라이웨이트 패턴이란?

> 플라이웨이트 패턴은 어떤 클래스의 인스턴스 하나로, 여러 개의 가상 인스턴스를 제공하고 싶을 때 사용합니다.
>
> 예를 들어, 나무를 1000개의 나무를 만들어서 위치값을 출력하는 로직이 있다고 했을 때, 1000의 나무를 만들어서 출럭하는것이 아닌, 1개의 나무를 만들어 놓고, 나무의 속성만 바꾸어서 출력하여 1개의 인스턴스로 1000개의 인스턴스가 생성된것 처럼 보이게 합니다. 


플라이웨이트 패턴은 재사용 가능한 객체 인스턴스를 공유시켜 메모리 사용량을 최소화 하는 패턴 입니다. 흔히 `Cache` 개념을 코드로 패턴화 한것으로 보면 됩니다.

즉, 동일하거나 유사한 객체들 사이에 가능한 많은 데이터를 서로 공유하여 사용하도록 하여 최적화를 노리는 `경량 패턴` 입니다.

### 패턴 사용 시기

- 어플리케이션에 의해 생성되는 객체의 수가 많아 저장비용이 높아질 때
- 생성된 객체가 오래도록 메모리에 상주하며 사용되는 횟수가 많을 때
- 공통적인 인스턴스를 많이 생성하는 로직이 포함되었을 때
- 임베디드와 같이 메모리를 최소한으로 사용해야 하는 경우

### 패턴 장점

- 애플리케이션에서 사용하는 메모리를 줄일 수 있음
- 이를통해, 프로그램 속도를 개선함

### 패턴 단점

- new 로 생성하는 로직보다 코드의 복잡성이 증가

## 예제

나무의 위치를 출력하는 프로그램을 만들어 봅니다. 

이 프로그램을 쉽게 짜려면, `Tree`라는 모델을 하나 만들어 놓고, new 키워드로 생성한 후에, 생성자로 위치를 넣어주고, 출력하는 함수를 만들어서 실행시켜주기만 하면 됩니다.

하지만 이렇게 되면, 나무가 많아질수록 메모리에 부담이 가게 되고, 성능이 저하될 수 있습니다.

그렇기에 Flyweight 패턴을 사용하여, 객체를 공유하고, 메모리의 부담을 줄여 봅니다.

### Tree 인터페이스 만들기

우선, 구체적인 Tree 를 만들기 전에, Tree 인터페이스를 만들어 상속을 시킬 준비를 합니다.

🗅 **<span style="color: #c03a92">interface Tree</span>**

```java
package src.flyweight;  
  
import java.time.LocalDate;  
import java.time.Month;  
  
public interface Tree {  
    public void display(int _x, int _y);  
  
    public default boolean isWithinRange(LocalDate _date) {  
        Month month =_date.getMonth();  
        return (month.getValue() > 2) && (month.getValue() < 11);  
    } 
}
```

### Tree를 상속받아 Tree 구현 클래스 만들기

인터페이스를 활용하여 Tree의 뼈대를 만들였으니, 이제 해당 인터페이스를 상속시켜 구체적인 Tree를 만들어 봅니다.

나무의 종류는 활엽수, 침엽수 로 정의 합니다.

🗅 **<span style="color: #c03a92">class DeciduousTree</span>**

```java
package src.flyweight;  
  
import java.time.LocalDate;  
  
public class DeciduousTree implements Tree {  
    @Override  
    public void display(int _x, int _y) {  
  
        System.out.println("활엽수 나무가 위치한 좌표: " + _x + ", " + _y);  
  
        if (!this.isWithinRange(LocalDate.now())) {  
            System.out.println("현재 나무에는 잎이 없습니다.");  
        }  
    }  
}
```

🗅 **<span style="color: #c03a92">class ConiferTree</span>**

```java
package src.flyweight;  
  
public class ConiferTree implements Tree {  
    @Override  
    public void display(int _x, int _y) {  
        System.out.println("침엽수 나무가 위치한 좌표" + _x + ", " + _y);  
    }  
}
```

### Tree를 생성할 Factory 만들기

Tree 의 생성 과정을 캡슐화 하는 `Factory`를 만듭니다. `SimpleFactory`로 구성하였습니다. 해당 `Factory` 는  Enum 값에 따라서 나무의 인스턴스를 반환 합니다.

🗅 **<span style="color: #c03a92">enum TreeType</span>**

```java
package src.flyweight;  
  
public enum TreeType {  
    Deciduous,  
    Conifer,  
}
```

🗅 **<span style="color: #c03a92">class TreeFactory</span>**

```java
package src.flyweight;  
  
public class TreeFactory {  
  
    private Tree deciduousTree = null;  
    private Tree coniferTree = null;  
  
    public TreeFactory() {  
        this.deciduousTree = new DeciduousTree();  
        this.coniferTree = new ConiferTree();  
    }  
  
    public Tree getTree(TreeType _type) throws Exception {  
        switch (_type) {  
            case Deciduous:  
                return this.deciduousTree;  
            case Conifer:  
                return this.coniferTree;  
            default:  
                throw new Exception("Invalid kind of tree");  
        }  
  
    }  
}
```

### Main 에서 실행

이제 준비가 되었으니, Main에서 실행해 봅니다.

여기서 봐야할 곳은 `for` 문이 돌아가는 곳입니다. `for` 문에서 `Tree` 객체를 생성하는 것이 아닌, `for`문이 돌기전에, 객체의 인스턴스를 하나 만들어 놓고, 그 객체에 위치 속성을 대입하여 출력하게 합니다.

🗅 **<span style="color: #c03a92">class Client</span>**

```java
package src.flyweight;  
  
public class Client {  
    public static void main(String[] args) {  
        int [][] deciduousLocations = { {1,1}, {33,50}, {100, 90} };  
        int [][] coniferLocations = { {10,87}, {24,76}, {2, 64} };  
  
        TreeFactory treeFactory = new TreeFactory();  
  
        Tree deciduous = null;  
        Tree conifer = null;  
  
        try{  
            deciduous = treeFactory.getTree(TreeType.Deciduous);  
            conifer = treeFactory.getTree(TreeType.Conifer);  
  
            for(int[] location : deciduousLocations){  
                deciduous.display(location[0], location[1]);  
            }  
  
            for(int[] location : deciduousLocations){  
                conifer.display(location[0], location[1]);  
            }  
        }  
        catch(Exception _ex){  
            _ex.printStackTrace();  
        }  
    }  
}
```

![](/images/Pasted%20image%2020241012233002.png)

### 정리

플라이웨이트 패턴을 사용하면서 Unity 엔진의 `Scriptable Object` 와 닮았다는 느낌을 받았습니다. Unity의 `Scriptable Object` 도 마찬가지로, 객체를 생성할 때, 공유하는 데이터를 하나의 객체에서 참조하는 형식으로 최적화를 하게 됩니다. 만약 공유하는 데이터가 10byte 라고 한다면, 10 x n 만큼의 메모리가 절약 되는 것이죠.

디자인 패턴을 공부하다 보면, 자연스럽게 "이걸 어디에 쓸 수 있을까?" 또는 "실제로 많이 사용하는 패턴일까?"라는 의문이 들곤 합니다. 다양한 프레임워크에서 이미 이런 패턴들이 구현되어 있다는 사실을 알게 될 때마다, 얼마나 유용하고 신기한지 다시금 느끼게 됩니다.