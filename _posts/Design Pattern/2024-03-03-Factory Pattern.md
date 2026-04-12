---
layout: post

title: "[Design Pattern] 팩토리 패턴 (Factory Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2023-03-02
last_modified_at: 2024-03-03
---

## 팩토리 패턴이란?

>팩토리 패턴(Factory Pattern)은 객체 지향 프로그래밍에서 객체를 생성하는 과정을 추상화 하는 디자인 패턴 입니다.
>
>이 패턴의 핵심 목적은 객체 생성 로직을 클라이언트 코드로부터 분리하여, 코드의 유연성과 재사용성을 높이는 것입니다.

팩토리패턴에는 다양한 형태가 있습니다.
- 간단한 팩토리 (Simple Factory)
- 팩토리 메소드 패턴(Factory Method Pattern)
- 추상 팩토리 패턴(Abstract Factory Pattern)

### <span style="color: #921c96">간단한 팩토리 (Simple Factory)</span>

간단한 팩토리(Simple Factory)는 디자인 패턴이라기 보다는 프로그래밍에서 자주 쓰이는 관용구에 가깝습니다. 하지만, 워낙 자주 쓰이기 때문에 알고 있어야 할 방식입니다.

#### 간단한 팩토리 장점
- 객체 생성 코드를 중앙 집중화 해서 관리할 수 있음 -> 생성 코드의 중복을 줄임
- 객체 생성 방법을 변경하려면, 팩토리 클래스만 수정하면 됨 -> 유지보수성 up

#### 간단한 팩토리 단점
- 객체 생성을 중앙 집중화 함으로, 팩토리 클래스가 비대해질 수 있음. 따라서 팩토리 클래스를 분리하거나, `팩토리 메서드 패턴`을 사용하는 것이 좋을 수 있음
- 객체 생성을 위한 인터페이스나, 추상 클래스가 없으므로, 유연성이 떨어짐

### <span style="color: #921c96">팩토리 메소드 패턴</span>

`Simple Factory`는 객체 생성 역할을 분리했지만, 새로운 클래스가 추가됐을 때, Factory 클래스를 수정해야 한다는 단점이 있었습니다.

팩토리 메소드 패턴에서는 객체를 생성할 때 필요한 인터페이스를 만듭니다. 어떤 클래스의 인스턴스를 만들지는 서브클래스에서 결정합니다. 

팩토리 메소드 패턴을 사용하면 클래스 인스턴스를 만드는 일을 서브클래스에서 맡게 됩니다.

#### 팩토리 메소드 패턴 장점
- 생성자(Creator)와 구현 객체(Concrete Product)의 강한 결합을 피함
- 객체의 생성 후 공통으로 할 일을 수행하도록 지정함
- 캡슐화, 추상화를 통해 생성되는 객체의 구체적인 타입을 감춤
- 단일 책임원칙, 개방 폐쇄 원칙 준수
- 생성에 대한 인터페이스와 구현 부분을 나누었으므로, 협업에 용이

#### 팩토리 패턴 단점
- 각 제품 구현체마다 팩토리 객체들을 모두 구현해주어야 함 -> 구현체가 늘어나면서 팩토리 클래스가 엄청 많아질 수 있음
- 코드의 복잡성 증가

### <span style="color: #921c96">추상 팩토리 패턴</span>

`Factory Method` 패턴에서는 객체지향의 원칙들을 준수하며, 인터페이스와 추상클래스를 활용하여 구현하였습니다.

하지만, 각 객체마다 필요한 또다른 객체가 필요할 수 있습니다.

예를들어, 피자를 만든다고 가정해 봅시다. 피자에는 여러가지의 재료들이 들어갑니다. `Factory Method`패턴에서는 해당 피자의 재료들을 각각 구상 클래스에서 정의해주어야 했습니다.

이렇게하면, 만들어야하는 객체가 늘어날 수록, 구상클래스를 변경해줘야 하기 때문에 유지보수성이 떨어집니다.

그럴때는, 추상 팩토리 패턴을 사용 합니다.

`추상 팩토리 패턴`은 연관성이 있는 객체 군이 여러개 있을 경우 이들을 묶어 추상화하고, 어떤 구체적인 상황이 주어지면 팩토리 객체에서 집합으로 묶은 객체 군을 구현화 하는 생성 패턴입니다.

## 팩토리 메서드 패턴 vs 추상 팩토리 패턴

둘다 팩토리 객체를 통해 구체적인 타입을 감추고, 객체 생성에 관여하는 패턴 입니다. 또한, 공장 클래스가 제품 클래스를 각각 나누어 느슨한 결합 구조를 구성하는 모습 또한 동일합니다.

`패토리 메서드 패턴`은 **객체 생성 이후, 해야할 일**의 공통점을 정의하는데 초점을 맞춘다면, `추상 팩토리 패턴`은 **생성해야 할 객체 집합 군**의 공통점에 초점을 맞춥니다.

그렇기 때문에, **추상 팩토리 패턴이 팩토리 메서드 패턴의 상위 호환은 아닙니다. 두 패턴의 차이는 명확하기 때문에, 상황에 따라 적절한 선택을 해야 합니다.**


![팩토리 메서드 vs 추상 팩토리](/images/Pasted%20image%2020240303180907.png)

표 출처: [https://inpa.tistory.com/entry/GOF-💠-추상-팩토리Abstract-Factory-패턴-제대로-배워보자#abstract_factory_vs_factory_method](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%B6%94%EC%83%81-%ED%8C%A9%ED%86%A0%EB%A6%ACAbstract-Factory-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90#abstract_factory_vs_factory_method) [Inpa Dev 👨‍💻:티스토리]


## 예제 - Simple Factory

피자가게에서 피자를 주문하는 과정을 Factory 패턴으로 구현해 보도록 합니다.

피자는 총 4가지 종류가 있습니다. (`Cheese`,`Pepperoni`,`Clam`,`Veggie`)

이 피자들 중 한가지를 고객이 주문하게 되면, 피자를 만들고, 자르고, 포장하여 고객에게 다시 전달합니다.

이번 예제에서는 `Cheese Pizza`를 주문해 봅니다.

우선, 피자들의 종류를 정의할 `Enum` 클래스를 만듭니다.

### 피자 종류 enum 클래스 정의

🗅 **<span style="color: #c03a92">enum Pizzas</span>**
```java
public enum Pizzas {  
    CHEESE,  
    PEPPERONI,  
    CLAM,  
    VEGGIE  
}
```

### 피자 추상 클래스 정의

그 후, 각 피자들을 아우르는 추상 클래스를 만들도록 합니다.

`Pizza`클래스는 각 피자의 이름/도우/소스/토핑들의 정보를 가지고 있고, 주문이 들어오면, `prepare()`->`bake()`-> `cut()`->`box()`의 과정을 거칩니다.

🗅 **<span style="color: #c03a92">abstract class Pizza</span>**
```java
public abstract class Pizza {  
  
    protected String name;  
    protected String dough;  
    protected String sauce;  
    List<String> toppings = new ArrayList<>();  
  
    public String getName() {  
        return name;  
    }  
  
    public void prepare() {  
        System.out.println("준비중입니다 " + name);  
    }  
    public void bake(){  
        System.out.println("요리중입니다 " + name);  
    }  
    public void cut(){  
        System.out.println("자르는중입니다 " + name);  
    }  
    public void box(){  
        System.out.println("포장중입니다 " + name);  
    }  
  
    @Override  
    public String toString(){  
        StringBuffer display = new StringBuffer();  
        display.append("---- " + name + " ----\n");  
        display.append(dough + "\n");  
        display.append(sauce + "\n");  
        for (String topping : toppings) {  
            display.append(topping + "\n");  
        }  
        return display.toString();  
    }  
  
}
```


### 피자 구상 클래스 정의

이제 각 피자들의 구상 클래스를 정의 합니다.

간단하게 `Cheese` 피자만 구현하도록 합니다.

해당 클래스에는 피자의 정보와 토핑들을 정의 합니다.

🗅 **<span style="color: #c03a92">class CheesePizza</span>**
```java
public class CheesePizza extends Pizza{  
    public CheesePizza() {  
        name = "치즈 피자";  
        dough = "보통 크러스트";  
        sauce = "Marinara Pizza Sauce";  
        toppings.add("Fresh Mozzarella");  
        toppings.add("Parmesan");  
    }  
}
```


### 팩토리 클래스 정의

이제 피자를 정의 했으니, 각 피자를 만드는 `Factory` 클래스를 정의 합니다.

피자의 타입을 받아서, 해당하는 피자 객체를 반환하는 역할을 합니다.

🗅 **<span style="color: #c03a92">class SimplePizzaFactory</span>**
```java
public class SimplePizzaFactory {  
    public Pizza createPizza(Pizzas _type) {  
        Pizza pizza = null;  
        switch (_type) {  
            case CHEESE:  
                pizza = new CheesePizza();  
                break;  
            case PEPPERONI:  
                pizza = new PepperoniPizza();  
                break;  
            case CLAM:  
                pizza = new ClamPizza();  
                break;  
            case VEGGIE:  
                pizza = new VeggiePizza();  
                break;  
        }  
        return pizza;  
    }   
}
```


### Pizza Store 정의

이제 주문을 받을 `Pizza Store`클래스를 정의 합니다. `PizzaStore` 에서는 `orderPizza()`라는 함수로 피자를 생성하는 명령을 내립니다.

🗅 **<span style="color: #c03a92">class PizzaStore</span>**
```java
public class PizzaStore {  
    SimplePizzaFactory factory;  
    public PizzaStore(SimplePizzaFactory _factory){  
        this.factory = _factory;  
    }  
  
    public Pizza orderPizza(Pizzas _type){  
        Pizza pizza;  
  
        pizza = factory.createPizza(_type);  
  
        pizza.prepare();  
        pizza.bake();  
        pizza.cut();  
        pizza.box();  
  
        return pizza;  
    }  
}
```

### Main 함수 실행

이제 준비가 되었으니, 피자를 주문해보도록 합니다.

`Pizza Store`를 만들고, `factory`를 주입해 줍니다.

그리고 `orderPizza()` 함수를 실행시켜 봅니다.

🗅 **<span style="color: #c03a92">class Main</span>**
```java
public class PizzaTestDrive {  
  
    public static void main(String[] args) {  
        SimplePizzaFactory factory = new SimplePizzaFactory();  
        PizzaStore store = new PizzaStore(factory);  
  
        Pizza pizza = store.orderPizza(Pizzas.CHEESE);  
        System.out.println(pizza.getName() + "을 주문 함" + "\n");  
        System.out.println(pizza);  
    }  
}
```

![실행결과](/images/Pasted%20image%2020240303183928.png)

요구 사항에 맞게 피자가 잘 만들어진 것을 볼 수 있습니다.

하지만, `Pizza Store`가 많아지고, `Pizza`의 종류가 많아지면, 결국 코드를 고쳐야 하는 단점이 존재합니다.

그렇기 때문에 간단한 객체를 만들때에 `Simple Factory`를 사용하게 됩니다.


## 예제 - Factory Method Pattern

이제 팩토리 메서드 패턴의 예제를 만들어 봅니다.

`Simple Factory`패턴으로 피자가 대중화 되어서, 이제 체인점을 내야 한다고 합니다.

이 체인점에는 각 지역별로 다른 피자를 팔게 됩니다.

기존의 `Simple Factory` 로 구현하려면, 각 체인점마다 `order()` 메서드가 달라져야 합니다. 또한, Pizzas 의 종류도 더 늘어나야겠죠.

이러한 문제를 해결하기 위해 `Factory Method Pattern`으로 구현합니다.

피자 가게를 추상화 시켜버리는 것이죠.

그렇게 되면, 각 `Pizza Store`에서 알아서 스타일 대로 피자를 만들어 줍니다.

즉, 피자의 종류를 NewYorkStyleCheesePizza, ChicagoStyleCheesePizza, SeoulStyleCheesePizza ... 등으로 나누지 않고, 해당 Pizza를 만들 수 있는 `PizzaStore` 에 의해 각자 알아서 만들어 지는 것이죠.
-> 피자의 종류는 어떤 서브클래스(어떤 Pizza Store)를 선택했느냐에 따라 결정 됩니다.

그렇다면, 이제 구현을 해봅시다.

Pizza Store 는 `NYPizzaStore` 와 `ChicagoStore`로 총 2가지 입니다.

### Pizza Store 추상클래스 만들기

Pizza Store 들을 아우르는 추상 클래스를 만듭니다.

🗅 **<span style="color: #c03a92">abstract class PizzaStore</span>**
```java
public abstract class PizzaStore {  
    public Pizza orderPizza(Pizzas _type){  
        Pizza pizza;  
        pizza = createPizza(_type);  
  
        pizza.prepare();  
        pizza.bake();  
        pizza.cut();  
        pizza.box();  
  
        return pizza;  
    }  
  
    //팩토리 객체 대신 이 메소드를 사용함  
    protected abstract Pizza createPizza(Pizzas _type);
}
```

`SimpleFactory`와의 다른점은 Factory 인터페이스를 각 서브클래스에서 DI 받게 된다는 것입니다.

### Pizza Store 구상클래스 만들기

이제 앞서 정의한 `Pizza Store`를 가지고 구상클래스들을 만들어 봅니다.
`NYPizzaStore`, `ChicagoPizzaStore` 총 2가지를 만들어 봅시다.

🗅 **<span style="color: #c03a92">class NYPizzaStore</span>**
```java
public class NYPizzaStore extends PizzaStore {  
    @Override  
    protected Pizza createPizza(Pizzas _type) {  
        Pizza pizza = null;  
        switch (_type) {  
            case CHEESE:  
                pizza = new NYStyleCheesePizza();  
            break;  
            case VEGGIE:  
                pizza = new NYStyleVeggiePizza();  
            break;  
            case CLAM:  
                pizza = new NYStyleClamPizza();  
            break;  
            case PEPPERONI:  
                pizza = new NYStylePepperoniPizza();  
            break;  
        }  
        return pizza;  
    }  
}
```

🗅 **<span style="color: #c03a92">class ChicagoPizzaStore</span>**
```java
public class ChicagoPizzaStore extends PizzaStore{  
  
    @Override  
    protected Pizza createPizza(Pizzas _type) {  
        Pizza pizza = null;  
        switch (_type) {  
            case CHEESE:  
                pizza = new ChicagoStyleCheesePizza();  
                break;  
            case VEGGIE:  
                pizza = new ChicagoStyleVeggiePizza();  
                break;  
            case CLAM:  
                pizza = new ChicagoStyleClamPizza();  
                break;  
            case PEPPERONI:  
                pizza = new ChicagoStylePepperoniPizza();  
                break;  
        }  
        return pizza;  
    }  
}
```

### Pizza 추상 클래스 구현

이제 만들어야 할 객체를 만들어 봅시다.

우선, Pizza들을 아우를 추상 Pizza 클래스를 만듭니다.

🗅 **<span style="color: #c03a92">abstract class Pizza</span>**
```java
public abstract class Pizza {  
    String name;  
    String dough;  
    String sauce;  
    ArrayList<String> toppings = new ArrayList<String>();  
  
    protected void prepare() {  
        System.out.println("준비중... " + name);  
        System.out.println("도우 전달중...");  
        System.out.println("소스 바르는 중...");  
        System.out.println("토핑 넣는 중: ");  
        for (String topping : toppings) {  
            System.out.println("   " + topping);  
        }  
    }  
  
    protected void bake() {  
        System.out.println("350도에서 25분 구움, bake 호출");  
    }  
  
    protected void cut() {  
        System.out.println("피자 자름, cut 호출");  
    }  
  
    protected void box() {  
        System.out.println("피자 포장중, box 호출");  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public String toString() {  
        StringBuffer display = new StringBuffer();  
        display.append("---- " + name + " ----\n");  
        display.append(dough + "\n");  
        display.append(sauce + "\n");  
        for (String topping : toppings) {  
            display.append(topping + "\n");  
        }  
        return display.toString();  
    }  
}
```

### 각 스타일에 맞는 피자 구현

이제 Pizza를 상속받아, 구체적인 피자 구상클래스를 만들도록 합니다.

지금은 총 4가지의 피자들이 있지만, 그중에서 각 스타일에 맞게 CheesePizza 를 만들어 보도록 합시다.

🗅 **<span style="color: #c03a92">class NYStyleCheesePizza</span>**
```java
public class NYStyleCheesePizza extends Pizza{  
    public NYStyleCheesePizza() {  
        name = "뉴욕 스타일 소스와 치즈 피자";  
        dough = "씬 크러스트 도우";  
        sauce = "마리나라 소스";  
  
        toppings.add("잘게 썬 레지아노 치즈");  
    }  
}
```

🗅 **<span style="color: #c03a92">class ChicagoStyleCheesePizza</span>**
```java
public class ChicagoStyleCheesePizza extends Pizza {  
  
    public ChicagoStyleCheesePizza() {   
       name = "시카고 스타일 딥 디쉬 치즈 피자";  
       dough = "아주 두꺼운 크러스트 도우";  
       sauce = "플럼토마토 소스";  
   
       toppings.add("잘게 조각낸 모짜렐라 치즈");  
    }  
   
    protected void cut() {  
       System.out.println("네모난 모양으로 피자 자르기");  
    }  
}
```

### Main 에서 실행

이제 준비가 모두 완료 되었으니, 피자를 만들어 봅시다.

🗅 **<span style="color: #c03a92">class ChicagoStyleCheesePizza</span>**
```java
public class Main {  
    public static void main(String[] args) {  
        PizzaStore nyPizzaStore = new NYPizzaStore();  
        Pizza pizza = nyPizzaStore.orderPizza(Pizzas.CHEESE);  
        System.out.println("에단이 주문한" + pizza.getName());  
  
        System.out.println("\n");  
  
        PizzaStore chicagoPizzaStore = new ChicagoPizzaStore();  
        pizza = chicagoPizzaStore.orderPizza(Pizzas.CHEESE);  
        System.out.println("조셉이 주문한" + pizza.getName());  
    }  
}
```

![](/images/Pasted%20image%2020240303210052.png)

각기 다른 사람이 주문한 각기 다른 스타일의 피자가 나온것을 볼 수 있습니다.

## 예제 - Abstract Factory Pattern

방금 구현한 `Factory Method Pattern`에서는 각 피자 가게와 피자들을 추상화해서 구현하였습니다.

피자를 구현할 때, 각 서브클래스에서 재료들을 정의했었습니다.

하지만, 단점으로는 피자들의 재료를 바꾸기 위해서는 각 서브클래스에 가서 스크립트를 수정해야 하는 것이 있습니다.

그것을 보완하기 위해서 추상 펙토리 패턴을 이용합니다.

재료들까지 추상화를 시켜버리는 것이죠.

### Pizza 재료 Factory 인터페이스 만들기

각 재료들을 반환하는 interface를 만듭니다.

그전에, 각 재료들의 클래스들을 만듭니다.

재료는 `Dough`, `Sauce`, `Cheese`, `Veggies`, `Pepperoni`, `Clams` 로 정의 합니다.

🗅 **<span style="color: #c03a92">class Dough</span>**

```java
public interface Dough {  
    public String toString();  
}
```

위 클래스와 같이 나머지 클래스들도 만들어 줍니다.

그 후, interface를 정의 합니다.

🗅 **<span style="color: #c03a92">interface PizzaIngredientFactory</span>**
```java
//피자의 재료를 만들기 위한 인터페이스  
public interface PizzaIngredientFactory {  
    public Dough createDough();  
    public Sauce createSauce();  
    public Cheese createCheese();  
    public Veggies[] createVeggies();  
    public Pepperoni createPepperoni();  
    public Clams createClam();  
}
```

### interface 상속받아서 각 스타일에 맞는 재료 공장 만들기

`NYPizzaIngredientFactory` 와 `ChicagoPizzaIngredientFactory` 를 만들어 줍니다.

그리고 `PizzaIngredientFactory` 인터페이스를 구상해 줍니다.

🗅 **<span style="color: #c03a92">interface NYPizzaIngredientFactory</span>**
```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory{  
    @Override  
    public Dough createDough() {  
        return new ThinCrustDough();  
    }  
  
    @Override  
    public Sauce createSauce() {  
        return new MarinaraSauce();  
    }  
  
    @Override  
    public Cheese createCheese() {  
        return new ReggianoCheese();  
    }  
  
    @Override  
    public Veggies[] createVeggies() {  
        Veggies[] veggies = {new Garlic(), new Onion(), new Mushroom(), new RedPepper()};  
        return veggies;  
    }  
  
    @Override  
    public Pepperoni createPepperoni() {  
        return null;  
    }  
  
    @Override  
    public Clams createClam() {  
        return null;  
    }  
}
```

🗅 **<span style="color: #c03a92">interface ChicagoPizzaIngredientFactory</span>**
```java
public class ChicagoPizzaIngredientFactory implements PizzaIngredientFactory {  
    @Override  
    public Dough createDough() {  
        return new ThickCrustDough();  
    }  
  
    @Override  
    public Sauce createSauce() {  
        return new PlumTomatoSauce();  
    }  
  
    @Override  
    public Cheese createCheese() {  
        return new MozzarellaCheese();  
    }  
  
    @Override  
    public Veggies[] createVeggies() {  
        Veggies[] veggies = {new BlackOlives(),new Spinach(), new Eggplant()};  
        return veggies;  
    }  
  
    @Override  
    public Pepperoni createPepperoni() {  
        return new SlicedPepperoni();  
    }  
  
    @Override  
    public Clams createClam() {  
        return new FrozenClams();  
    }  
}
```

### Pizza 클래스 변경하기

Pizza 클래스에 새로 추가된 재료들을 담을 변수들을 선언해 줍니다.

또한, `prepare()` 메서드를 추상 메서드로 바꿔 줍니다. 서브클래스에서 `prepare()` 메서드를 재정의 하여 필요한 재료들을 가져오도록 합니다.

🗅 **<span style="color: #c03a92">abstract class Pizza</span>**
```java
public abstract class Pizza {  
  
    protected String name;  
    protected Dough dough;  
    protected Sauce sauce;  
    protected Veggies[] veggies;  
    protected Cheese cheese;  
    protected Pepperoni pepperoni;  
    protected Clams clam;  
  
    abstract void prepare();  
  
    protected void bake(){  
        System.out.println("175도에서 25분 간 굽기");  
    }  
    protected void cut(){  
        System.out.println("피자 사선으로 자르기");  
    }  
    protected void box(){  
        System.out.println("상자에 피자 담기");  
    }  
    protected void setName(String _name){  
        this.name = _name;  
    }  
    protected String getName(){  
        return name;  
    }  
    public String toString(){  
        //피자 이름 출력  
        return null;  
    }  
}
```

### Pizza 서브클래스 정의

이제 Pizza 클래스를 상속받아 구상클래스를 만들어 줍니다.

`CheesePizza` 를 예로들어 구상해 봅시다.

🗅 **<span style="color: #c03a92">abstract class CheesePizza</span>**
```java
public class CheesePizza extends Pizza{  
  
    PizzaIngredientFactory ingredientFactory = null;  
  
    public CheesePizza(PizzaIngredientFactory _ingredientFactory){  
        this.ingredientFactory = _ingredientFactory;  
    }  
  
    @Override  
    void prepare() {  
        System.out.println("준비중"+ name);  
        dough = ingredientFactory.createDough();  
        sauce = ingredientFactory.createSauce();  
        cheese = ingredientFactory.createCheese();  
    }  
}
```

`prepare()` 메서드에서 펙토리에서 가져올 재료들을 매핑시켜 줍니다.

### PizzaStore 구상

PizzaStore 추상 클래스는 그대로 사용하고, 구상클래스를 살짝 바꿔줍니다.

🗅 **<span style="color: #c03a92">abstract class NYPizzaStore</span>**
```java
public class NYPizzaStore extends PizzaStore {  
    @Override  
    protected Pizza createPizza(Pizzas _item) {  
        Pizza pizza = null;  
        PizzaIngredientFactory ingredientFactory = new NYPizzaIngredientFactory();  
  
        switch (_item) {  
            case CHEESE:  
                pizza = new CheesePizza(ingredientFactory);  
                pizza.setName("뉴욕 스타일 치즈 피자");  
                break;  
            case VEGGIE:  
                pizza= new VeggiePizza(ingredientFactory);  
                pizza.setName("뉴욕 스타일 야채 피자");  
                break;  
            case CLAM:  
                pizza = new ClamPizza(ingredientFactory);  
                pizza.setName("뉴욕 스타일 조개 피자");  
                break;  
            case PEPPERONI:  
                pizza = new PepperoniPizza(ingredientFactory);  
                pizza.setName("뉴욕 스타일 페퍼로니 피자");  
                break;  
        }  
        return pizza;  
    }  
}
```

치즈피자에 `IngredeintFactory`를 DI해 줍니다.

### Main 에서 실행

각기 다른 CheesePizza 를 만들어 봅시다.

🗅 **<span style="color: #c03a92">class Main</span>**
```java
public class Main {  
    public static void main(String[] args) {  
        PizzaStore nyStore = new NYPizzaStore();  
        PizzaStore chicagoStore = new ChicagoPizzaStore();  
  
        Pizza pizza = nyStore.orderPizza(Pizzas.CHEESE);  
        printPizzaStatus("A",pizza);  
  
        pizza = chicagoStore.orderPizza(Pizzas.CHEESE);  
        printPizzaStatus("B",pizza);  
    }  
  
    private static void printPizzaStatus(String name,Pizza pizza){  
        System.out.println(name+ " ordered a " + pizza.name + "\n");  
        System.out.println(name+ " ordered a " + pizza.dough + "\n");  
        System.out.println(name+ " ordered a " + pizza.sauce + "\n");  
        System.out.println(name+ " ordered a " + Arrays.toString(pizza.veggies) + "\n");  
    }  
}
```

![](/images/Pasted%20image%2020240303213744.png)