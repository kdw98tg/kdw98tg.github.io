---
title: "[Design Pattern] 템플릿 메소드 패턴 (Templete Method Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2024-09-07
last_modified_at: 2024-09-07
---
## 템플릿 메소드 패턴(Templete Method Pattern) 이란?

> 템플릿 메소드 패턴이란, 여러 클래스에서 공통으로 사용하는 메서드를 템플릿화 하여 상위 클래스에서 정의하고, 하위클래스마다 세부 동작을 구현하는 패턴입니다.
>
>즉, 변하지 않는 기능은 상위 클래스에서 정의하고, 변하는 기능만 하위 클래스에서 구현하여 사용하는 것입니다.
>
>어떻게 보면, 알고리즘 군의 틀을 만들어 놓고, 다양한 알고리즘을 하위 클래스에서 재정의 한다는 전략 패턴과 유사한 점이 있지만, 템플릿 메소드 패턴은 전략패턴과 다르게 알고리즘의 순서를 정의하여 놓는다는 점에서 차이점이 있습니다.
>
>또한, Hook 메서드라는 것을 두어, 정의된 흐름을 제어할 수 있는 기능도 있습니다.

### 패턴의 사용 시기

- 정해진 순서대로 알고리즘이 돌아가야 하나, 각 클래스마다 정의가 다를 때

### 패턴 장점

- 알고리즘의 특정 부분만 재정의 하여, 알고리즘의 다른 부분에 발생하는 변경 사항의 영향을 덜 받게 함
- 코드의 중복을 줄임

### 패턴 단점

- 알고리즘의 제공된 골격에 의해 유연성이 제한될 수 있음
- 상위 클래스에서 수정이 발생하면, 하위의 모든 클래스가 영향을 받음


## 예제

음료를 제작하는 과정을 템플릿 메서드로 정의해 봅시다. 

우선 음료를 만들때는 `물을 끓인다 -> 우려낸다 -> 컵에 따른다 -> 첨가물을 넣는다`의 순서로 진행을 한다고 가정을 합니다. 순서는 같은데, 어떤 행동을 할지는 각 음료마다 다르기 때문에, 순서와 중복되는 행동들을 상위클래스에서 정의해 봅시다.

### 템플릿 제작

위에서 가정한 순서에 맞게, `prepareRecipe`라는 메소드를 정의 합니다.

🗅 **<span style="color: #c03a92">abstract class CaffeineBeverage</span>**

```java
public abstract class CaffeineBeverage {  
    final void prepareRecipe(){  
        boilWater();  
        brew();  
        pourInCup();  
        addCondiments();  
    }  
    protected abstract void brew();  
    protected abstract void addCondiments();  
    private void boilWater(){  
        System.out.println("물 끓이는 중");  
    }  
    private void pourInCup(){  
        System.out.println("컵에 따르는 중");  
    }  
}
```

### 하위 구현 클래스 제작

이제 상위 클래스를 정의 하였으니, 구체적인 행동을 정의할 하위 클래스를 정의 해 봅시다. 종류는 `Coffee`와 `Tea`로 구성합니다.

상위 클래스의 `brew()`와 `addCondiments()` 를 재정의 합니다.

🗅 **<span style="color: #c03a92"> class Coffee</span>**

```java
package src.templeteMethod.classicTemplate;  
  
public class Coffee extends CaffeineBeverage{  
      
    @Override  
    protected void brew() {  
        System.out.println("필터로 커피를 우려내는 중");      
    }  
  
    @Override  
    protected void addCondiments() {  
        System.out.println("설탕과 우유를 추가하는 중");  
    }  
}
```


🗅 **<span style="color: #c03a92"> class Tea</span>**
```java
package src.templeteMethod.classicTemplate;  
  
public class Tea extends CaffeineBeverage{  
    @Override  
    protected void brew() {  
        System.out.println("찻잎을 우려내는 중");  
    }  
  
    @Override  
    protected void addCondiments() {  
        System.out.println("레몬을 추가하는 중");  
    }  
}
```

이렇게 하면, 순서는 같지만, `brew()`라는 행동과 `addCondiments()`라는 행동을 각각 정의한 음료가 탄생하게 됩니다.

이제, 상위클래스에서 정의한 `prepareRecipe()`함수를 호출하면, 하위 클래스에서 재정의 한 메서드가 호출되며, 각 음료의 특성에 맞게 음료를 제작하게 됩니다.

### Main 에서 실행

Main 클래스에서 위에서 만든 음료 생성 과정을 실행해 봅니다.

🗅 **<span style="color: #c03a92"> class Tea</span>**

```java
package src.templeteMethod.classicTemplate;  
  
public class Main {  
    public static void main(String[] args) {  
        CaffeineBeverage tea = new Tea();  
        tea.prepareRecipe();  
  
        System.out.println("\n");  
  
        CaffeineBeverage coffee = new Coffee();  
        coffee.prepareRecipe();  
    }  
}
```


![Run Main](/images/Pasted%20image%2020240907165414.png)

위와같이 순서는 동일하되, 만드는 방법만이 다른 음료가 탄생하였습니다.

### Hook Method 이용

하지만, 음료마다 특정 조건이 필요할 수 있습니다. 예를들어, 어떤 음료는 사용자가 첨가물을 원할때만 첨가물을 넣어야 한다 라는 상황입니다.

이럴때, 템플릿의 흐름을 제어하는 용도로 사용하는것이 `Hook Method` 입니다.

`Tea` 라는 음료는 사용자의 동의가 있어야, 첨가물을 넣을 수 있다고 가정해 봅시다.

🗅 **<span style="color: #c03a92">abstract class CaffeineBeverageWithHook</span>**

```java
package src.templeteMethod.classicTemplate;  
  
public abstract class CaffeineBeverageWithHook {  
    final void prepareRecipe(){  
        boilWater();  
        brew();  
        pourInCup();  
        if(customerWantsCondiments())  
            addCondiments();  
    }  
  
    protected abstract void brew();  
    protected abstract void addCondiments();  
    private void boilWater(){  
        System.out.println("물 끓이는 중");  
    }  
    private void pourInCup(){  
        System.out.println("컵에 따르는 중");  
    }  
    public boolean customerWantsCondiments(){  
        return true;  
    }  
}
```

아까 정의하였던, `CaffeineBeverage`와 거의 동일하지만, `customerWantsCondiments()`라는 함수가 추가되어서, 흐름을 제어하고 있는 것을 볼 수 있습니다.

### Hook 음료 구현

이제 Hook 메서드가 구현된 상위 클래스를 상속받아, 하위 클래스를 구현해 봅니다.

```java
package src.templeteMethod.classicTemplate;  
  
import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStreamReader;  
  
public class TeaWithHook extends CaffeineBeverageWithHook{  
    @Override  
    protected void brew() {  
        System.out.println("찻잎을 우려내는 중");  
    }  
  
    @Override  
    protected void addCondiments() {  
        System.out.println("레몬을 추가하는 중");  
    }  
  
    @Override  
    public boolean customerWantsCondiments(){  
        String answer = getUserInput();  
        if(answer.toLowerCase().startsWith("y")){  
            return true;  
        }  
        else{  
            return false;  
        }  
    }  
  
    private String getUserInput() {  
        String answer = null;  
  
        System.out.print("차에 레몬을 넣으시겠습니까? (y/n)? ");  
  
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));  
        try {  
            answer = in.readLine();  
        } catch (IOException ioe) {  
            System.err.println("IO error trying to read your answer");  
        }  
        if (answer == null) {  
            return "no";  
        }  
        return answer;  
    }  
}
```

마찬가지로, 아까 정의했던 `Tea` 음료와 흡사하지만, `customerWantsCondiments()` 라는 함수를 재정의 해준다는 점에서 차이가 있습니다.

해당 클래스를 정의하고 `Main`에서 실행해 봅니다.

```java
package src.templeteMethod.classicTemplate;  
  
public class Main {  
    public static void main(String[] args) {    
        CaffeineBeverageWithHook teaWithHook = new TeaWithHook();  
        teaWithHook.prepareRecipe();  
    }  
}
```

![](/images/Pasted%20image%2020240907165954.png)

실행 후, 살펴보면, 아까와 다른점은 `차에 레몬을 넣겠냐?` 라는 사용자의 의사를 물어보고 있다는 점입니다. 

y 라고 답했기 때문에, 레몬을 넣는 모습입니다. 반면, y 가 아니라 다른 답이 들어가면, 상위 클래스에서 정의한 대로 if(false)가 되어, 첨가물을 넣는 로직은 돌아가지 않게 됩니다.

![](/images/Pasted%20image%2020240907170134.png)

## 디자인 원칙

예제를 통해서, 템플릿 메소드를 구현해 보았습니다. 여기서 새로운 디자인 원칙을 하나 소개합니다.

바로 할리우드 원칙 입니다.
>할리우드 원칙 이란 상위 클래스에서 먼저 연락을 돌린다는 뜻입니다. 즉, 하위클래스에서 상위 클래스로 연락을 하지 말자! 라는 원칙 입니다.

이것을 통해 `의존성 부패`를 방지할 수 있다고 합니다. 의존성 부패란 어떤 고수준 구성 요소가 저수준 구성 요소에 의존하고, 그 저수준 구성 요소는 다시 고수준 구성 요소에 의존하고, 그 고수준 구성요소는 다시 또 다른 구성요소에 의존하고, 그 다른 구성요소는 또 저수준 구성 요소에 의존하는 것과 같은 식으로 의존성이 복잡하게 꼬여있는 상황을 뜻합니다.

이렇게 되어있으면, 시스템이 어떻게 설계되었는지 파악하기가 완전 힘들게 됩니다. 

할리우드 원칙을 사용하면, 저수준의 구성요소를 고수준의 구성요소가 호출 하게 됩니다. 즉, 상위 구성요소가 먼저 연락할 때 까지, 저수준의 구성요소는 상위 객체로 연락하지 말라! 라고 하는 뜻인겁니다.


