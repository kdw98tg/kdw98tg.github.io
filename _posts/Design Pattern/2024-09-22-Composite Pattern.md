---
layout: post

title: "[Design Pattern] 컴포지트 패턴 (Composite Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2024-09-22
last_modified_at: 2024-09-22
---

## 컴포지트 패턴 (Composite Pattern) 이란?

> 컴포지트 패턴 이란 복합객체(Composite) 와 단일객체(Leaf)를 동일한 컴포넌트로 취급하여 이 둘을 구분하지 않고 동일한 인터페이스를 사용하도록 하는 패턴입니다.
>
> Iterator 패턴에서 각 노드의 자식 객체도 순회해야 할 때 이 패턴을 사용할 수 있습니다.
>
> 또한, 객체 사이들의 관계를 트리계층 구조로 정의해야 할때 유용 합니다.
> 
> 즉, Composite 패턴은 그릇과, 내용물을 동일시해서 재귀적인 구조를 만들기 위한 패턴 입니다.

### 구조 및 용어
![](/images/Pasted%20image%2020240922223628.png)

- Component : Leaf와 Compsite를 묶는 공통적인 상위 인터페이스
- Composite : 복합 객체로서, Leaf 역할이나 Composite 역할을 넣어 관리하는 역할을 함
- Leaf : 단일 객체로서, 단순하게 내용물을 표시하는 역할을 함

### 패턴의 사용 시기

- 데이터를 다룰 때 계층적 트리 표현을 다루어야 할 때
- 복잡하고 난해한 단일/복합 객체 관계를 간편히 단순화 하여 균일하게 처리하고 싶을 때


### 패턴 장점

- 단일체와 복합체를 동일하게 여기기 때문에, 묶어서 계산하기 좋음
- 재귀를 통해 복잡한 트리구조를 보다 편하게 구성할 수 있음


### 패턴 단점

- 트리의 깊이가 깊어질 수록 디버깅이 어려워짐
- 아무 객체나 같은 객체로 취급하기 때문에, 예외처리하기가 힘듦

## 예제

이번에 살펴볼 예제는 `Iterator Pattern` 에서 다루었던 예제와 비슷합니다. 모든 메뉴들을 순회하는 예제 입니다.

하지만 `Iterator Pattern`에서 살펴본 예제와 다른 점은, 어떤 메뉴 안에 자식 메뉴가 들어있다는 것입니다.

즉, 계층이 1개가 아니라 2개 이상이 되는 경우 입니다.

### Component 클래스 생성

우선, 모든 노드가 상속받을 하나의 객체를 만들어 줍니다. 해당 객체의 이름은 `MenuComponent` 입니다.

이중, 어떤 메소드는 MenuItem 에서만 쓸 수 있고, 또 어떤 메소드는 Menu 에서만 쓸 수 있기에 기본적으로 모두 UnsupportedOperationException을 던지도록 했습니다.

이러면 자기 역할에 맞지 않는 메소드는 오버라이드 하지 않고, 기본 구현을 그대로 사용할 수 있습니다.

🗅 **<span style="color: #c03a92">class MenuComponent</span>**
```java
package src.composite.menu;  
  
public abstract class MenuComponent {  
//////////////////////////////////////////////////////////////////////////
	//MenuComponent 를 추가하거나 제거하는 메소드
    public void add(MenuComponent _menuComponent) {  
        throw new UnsupportedOperationException();  
    }  
    public void remove(MenuComponent _menuComponent) {  
        throw new UnsupportedOperationException();  
    }  
    public MenuComponent getChild(int _i) {  
        throw new UnsupportedOperationException();  
    }  
//////////////////////////////////////////////////////////////////////////

//////////////////////////////////////////////////////////////////////////
	//MenuItem 에서 작업을 처리하는 메소드
    public String getName() {  
        throw new UnsupportedOperationException();  
    }  
    public String getDescription() {  
        throw new UnsupportedOperationException();  
    }  
    public double getPrice() {  
        throw new UnsupportedOperationException();  
    }  
    public boolean isVegetarian() {  
        throw new UnsupportedOperationException();  
    }  
    public void print() {  
        throw new UnsupportedOperationException();  
    }  
//////////////////////////////////////////////////////////////////////////
}
```

### MenuComponent를 상속받는 MenuItem 만들기

이제 `MenuItem`을 만들어서 컬렉션에 들어갈 형태를 정의 합니다.

```java
package src.composite.menu;  
  
public class MenuItem extends MenuComponent {  
    private String name = null;  
    private String description = null;  
    private boolean isVegetarian = false;  
    private double price = 0d;  
  
    public MenuItem(String _name, String _description, boolean _isVegetarian, double _price) {  
        this.name = _name;  
        this.description = _description;  
        this.isVegetarian = _isVegetarian;  
        this.price = _price;  
    }  
  
    @Override  
    public String getName() {  
        return name;  
    }  
    @Override  
    public String getDescription() {  
        return description;  
    }  
    @Override  
    public double getPrice() {  
        return price;  
    }  
    @Override  
    public boolean isVegetarian() {  
        return isVegetarian;  
    }  
    @Override  
    public void print() {  
        System.out.print("  " + getName());  
        if (isVegetarian()) {  
            System.out.print("(v)");  
        }  
        System.out.println(", " + getPrice());  
        System.out.println("     -- " + getDescription());  
    }  
}
```


### MenuComponent 를 상속받는 Menu 만들기

이제 복합 객체 클래스인 Menu를 만듭니다.
해당 클래스는 `MenuItem`을 리스트 형태로 관리 합니다.

```java
package src.composite.menu;  
  
import java.util.ArrayList;  
import java.util.Iterator;  
  
public class Menu extends MenuComponent{  
    private ArrayList<MenuComponent> menuComponents = new ArrayList<MenuComponent>();  
    private String name = null;  
    private String description = null;  
  
    public Menu(String _name, String _description){  
        this.name = _name;  
        this.description = _description;  
    }  
  
    public void add(MenuComponent _menuComponent){  
        menuComponents.add(_menuComponent);  
    }  
    public void remove(MenuComponent _menuComponent){  
        menuComponents.remove(_menuComponent);  
    }  
  
    public MenuComponent getChild(int _i){  
        return (MenuComponent)menuComponents.get(_i);  
    }  
  
    public String getName(){  
        return name;  
    }  
    public String getDescription(){  
        return description;  
    }  
    public void print() {  
        System.out.print("\n" + getName());  
        System.out.println(", " + getDescription());  
        System.out.println("---------------------");  
  
        for (MenuComponent menuComponent : menuComponents) {  
            menuComponent.print();  
        }  
    }  
}
```

### Menu 에 들어있는 MenuItem 을 읽어줄 Waitress 클래스 만들기

이제 메뉴와 그 안에 들어갈 아이템들을 정의 하였으니, 그것들을 순환하면서 읽어줄 `Waitress`클래스를 정의 합니다.

해당 클래스는 메뉴들이 담겨있는 MenuComponent 를 받아서 그 안에 구현되어있는 `print()`메서드를 호출하여, 모든 요소들을 순회하도록 합니다.

```java
package src.composite.menu;  
  
public class Waitress {  
    private MenuComponent allMenus = null;  
    public Waitress(MenuComponent _allMenus){  
        this.allMenus = _allMenus;  
    }  
    public void printMenu(){  
        allMenus.print();  
    }  
}
```

### Main 클래스 정의

메인 클래스에서 각 컴포넌트들을 생성한 후, MenuItem 들을 밀어넣고, Waitress 를 생성하여 `printMenu()` 함수를 호출 합니다.

1번째 계층으로는 `pancakeHouseMenu`, `dinerMenu`, `cafeMenu`를 정의 합니다.

2번째 계층으로는 `dinerMenu` 의 자식 노드로 `dessertMenu` 를 정의 합니다.
2번째 계층으로는 `cafeMenu` 의 자식 노드로 `coffeeMenu` 를 정의 합니다.

이렇게 구성한 후, `Waitress` 를 생성하여 `printMenu()` 함수를 호출 합니다. 이전과 다르게, 각 `Iterator`마다 `Waitress`의 생성자로 넣어줄 필요 없이, `MenuComponent`의 형태인 `allMenu` 하나만 넣어주면 됩니다.

```java
package src.composite.menu;  
  
public class Main {  
    public static void main(String[] _args) {  
        MenuComponent pancakeHouseMenu = new Menu("팬케이크 하우스 메뉴", "아침식사");  
        MenuComponent dinerMenu = new Menu("다이너 메뉴", "점심식사");  
        MenuComponent cafeMenu = new Menu("카페 메뉴", "저녁식사");  
        MenuComponent dessertMenu = new Menu("디저트 메뉴", "당연히 디저트!");  
        MenuComponent coffeeMenu = new Menu("커피 메뉴", "오후 커피와 함께할 메뉴");  
  
        MenuComponent allMenus = new Menu("모든 메뉴", "모든 메뉴를 합친 것");  
  
        allMenus.add(pancakeHouseMenu);  
        allMenus.add(dinerMenu);  
        allMenus.add(cafeMenu);  
  
        pancakeHouseMenu.add(new MenuItem(  
                "K&B의 팬케이크 아침식사",  
                "스크램블 에그와 토스트를 곁들인 팬케이크",  
                true,  
                2.99));  
        pancakeHouseMenu.add(new MenuItem(  
                "일반 팬케이크 아침식사",  
                "프라이드 에그와 소시지를 곁들인 팬케이크",  
                false,  
                2.99));  
        pancakeHouseMenu.add(new MenuItem(  
                "블루베리 팬케이크",  
                "신선한 블루베리와 블루베리 시럽을 넣은 팬케이크",  
                true,  
                3.49));  
        pancakeHouseMenu.add(new MenuItem(  
                "와플",  
                "블루베리 또는 딸기를 선택할 수 있는 와플",  
                true,  
                3.59));  
  
        dinerMenu.add(new MenuItem(  
                "채식주의자 BLT",  
                "베이컨 대신 채식 베이컨을 사용한 상추와 토마토 샌드위치",  
                true,  
                2.99));  
        dinerMenu.add(new MenuItem(  
                "BLT",  
                "베이컨과 상추, 토마토를 곁들인 샌드위치",  
                false,  
                2.99));  
        dinerMenu.add(new MenuItem(  
                "오늘의 스프",  
                "감자 샐러드를 곁들인 오늘의 스프",  
                false,  
                3.29));  
        dinerMenu.add(new MenuItem(  
                "핫도그",  
                "사워크라우트, 렐리쉬, 양파, 치즈를 얹은 핫도그",  
                false,  
                3.05));  
        dinerMenu.add(new MenuItem(  
                "찐 야채와 현미",  
                "현미 위에 찐 야채를 얹은 요리",  
                true,  
                3.99));  
  
        dinerMenu.add(new MenuItem(  
                "파스타",  
                "마리나라 소스를 곁들인 스파게티와 사워도우 빵 한 조각",  
                true,  
                3.89));  
  
        dinerMenu.add(dessertMenu);  
  
        dessertMenu.add(new MenuItem(  
                "애플파이",  
                "바삭한 크러스트와 바닐라 아이스크림을 얹은 애플파이",  
                true,  
                1.59));  
  
        dessertMenu.add(new MenuItem(  
                "치즈케이크",  
                "초콜릿 그레이엄 크러스트를 곁들인 뉴욕 스타일 크림치즈 케이크",  
                true,  
                1.99));  
        dessertMenu.add(new MenuItem(  
                "셔벗",  
                "라즈베리 셔벗과 라임 셔벗 한 스쿱씩",  
                true,  
                1.89));  
  
        cafeMenu.add(new MenuItem(  
                "베지 버거와 에어프라이드 감자튀김",  
                "통밀 빵에 상추, 토마토를 넣은 베지 버거와 감자튀김",  
                true,  
                3.99));  
        cafeMenu.add(new MenuItem(  
                "오늘의 스프",  
                "샐러드를 곁들인 오늘의 스프",  
                false,  
                3.69));  
        cafeMenu.add(new MenuItem(  
                "부리또",  
                "전체 통 팥, 살사, 과카몰리를 넣은 큰 부리또",  
                true,  
                4.29));  
  
        cafeMenu.add(coffeeMenu);  
  
        coffeeMenu.add(new MenuItem(  
                "커피 케이크",  
                "시나몬과 호두가 올라간 바삭한 케이크",  
                true,  
                1.59));  
        coffeeMenu.add(new MenuItem(  
                "베이글",  
                "참깨, 양귀비씨, 시나몬 건포도, 호박 맛 중 선택 가능",  
                false,  
                0.69));  
        coffeeMenu.add(new MenuItem(  
                "비스코티",  
                "아몬드 또는 헤이즐넛 비스코티 세 개",  
                true,  
                0.89));  
  
        Waitress waitress = new Waitress(allMenus);  
  
        waitress.printMenu();  
    }  
}
```

![](/images/Pasted%20image%2020240922230235.png)

![](/images/Pasted%20image%2020240922230255.png)

위 사진과 같이 모든 자식의 노드까지 순회한 것을 알 수 있습니다.

## 결론

`MenuComponent`를 보면, `MenuComponent`를 관리하는 메서드와 메뉴 아이템을 관리하는 메서드 두가지 역할을 담당하고 있습니다.

이는 객체지향 원칙 중 `OCP` 원칙을 위배하고 있습니다.

그러나, `MenuComponent`의 설계는 투명성을 제공합니다. 

여기서 투명성이란, `Component` 인터페이스에 자식 요소를 관리하는 기능과 개별 요소(Leaf)로서의 기능을 모두 포함시켜, 클라이언트가 복합 객체(Composite)와 잎(Leaf)을 동일한 방식으로 처리할 수 있도록 만드는 것입니다. 이렇게 하면 클라이언트는 객체의 구체적인 타입을 신경 쓰지 않고도 일관된 방식으로 객체를 사용할 수 있습니다.

이는 객체지향 원칙을 상황에 따라 적절하게 사용해야 한다는것을 보여주는 대표적인 예 입니다.

