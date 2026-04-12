---
layout: post

title: "[Design Pattern] 이터레이터 패턴 (Iterator Pattern)"

categories:
  -  Architecture&Engineering
  
tags:
  - Java
  - Head-First-Design-Pattern
  - Book

toc: true
toc_sticky: true

date: 2024-09-16
last_modified_at: 2024-09-16
---

## 반복자 패턴 (Iterator Pattern) 이란?

> 반복자 패턴 이란  일련의 데이터 집합에 대하여 순차적인 접근을 지원하는 패턴 입니다. 이 과정에서 집합의 구현 방법을 노출하지 않아도 된다는 특징이 있습니다.
>
>foreach 구문으로 컬렉션들을 모두 순환시킬 수 있는 이유가 컬렉션들이 모두 Iterator 패턴으로 구현되어 있기 때문 입니다.


### 패턴의 사용 시기

- 컬렉션의 종류에 상관 없이 객체 접근 순회 방식을 통일하고자 할 때
- 컬렉션을 모두 순회하고 싶을 때

### 패턴 장점

- 일관된 인터페이스를 사용하여 모든 컬렉션 요소를 순회할 방법을 제시함
- 컬렉션의 내부 구조 및 순회 방법을 사용하는 곳에서 몰라도 됨
- SRP(단일책임원칙) 과 OCP(개방 폐쇄 원칙)을 준수 할 수 있음

### 패턴 단점

- 클래스가 늘어나고 복잡도가 증가함
- 구현 방법에 따라서 캡슐화를 위배할 수 있음

## 예제

Iterator 패턴은 어떤한 컬렉션에서 모든 요소들을 순회하고 싶을 때 사용합니다.

이번 예제에서는 아침메뉴와 점심메뉴를 웨이터가 모두 순회하며 소개하는 상황을 가정합니다.

또한, 각 메뉴들을 담을 컬렉션의 종류도 다르다고 가정을 합니다. `DinerMenu` 는 배열로 구성되어 있고, `PancakeHouseMenu`는 자바의 리스트로 구성되어 있습니다.

각 메뉴의 컬렉션 구조에 해당하는 `Iterator` 구현 클래스를 만들어서 모든 요소를 돌아보며 `Describe` 하는 로직입니다.

### 구상 클래스 만들기

우선 구상 클래스부터 정의를 합니다. 아직 어떤 방식으로 순회할지는 모르지만, `Iterator` 라는 인터페이스를 만들어서, 순회할 수 있도록 함수를 정의 합니다.

🗅 **<span style="color: #c03a92">interface Iterator</span>**

```java
package src.iterator.dinerMerger;  
  
public interface Iterator{  
    public boolean hasNext();  
    public MenuItem next();  
}
```

그리고, 메뉴 아이템들은 모두 Iterator 를 만들어야 하기 때문에, `Menu` 라는 인터페이스를 만들어서 Iterator 를 생성하는 과정을 정의 합니다.

🗅 **<span style="color: #c03a92">interface Menu</span>**

```java
package src.iterator.dinerMerger;  
  
public interface Menu {  
    public Iterator createIterator();  
}
```

이제는, 배열 또는 리스트의 요소로 담길 Model 을 만들어 줍니다.

🗅 **<span style="color: #c03a92">class MenuItem</span>**

```java
package src.iterator.dinerMerger;  
  
public class MenuItem {  
    private String name;  
    private String description;  
    private boolean vegetarian;  
    private double price;  
  
    public MenuItem(String _name, String _description, boolean _vegetarian, double _price) {  
        this.name = _name;  
        this.description = _description;  
        this.vegetarian = _vegetarian;  
        this.price = _price;  
    }  
  
    public String getName() {  
        return this.name;  
    }  
  
    public String getDescription() {  
        return this.description;  
    }  
  
    public double getPrice() {  
        return price;  
    }  
  
    public boolean isVegetarian() {  
        return vegetarian;  
    }  
  
    public String toString() {  
        return (name + ", $" + price + "\n   " + description);  
    }  
}
```


### 각 메뉴들이 담겨있는 클래스 만들기

이제 구상클래스들을 다 정의 하였으니, 실제로 메뉴들의 정보를 담고 있는 클래스들을 만들어 봅니다.

생성자에서 MaxItem 의 갯수만큼 배열의 길이를 정해주고, 아이템들을 만들어서 배열에 집어 넣습니다.

또한 Menu 인터페이스를 상속받아 `createIterator()` 라는 함수를 구현해 줍니다. 

🗅 **<span style="color: #c03a92">class DinerMenu</span>**

```java
package src.iterator.dinerMerger;  
  
public class DinerMenu implements Menu {  
  
    private static final int MAX_ITEMS = 6;  
    private int numberOfItems = 0;  
    private MenuItem[] menuItems;  
  
    public DinerMenu() {  
        menuItems = new MenuItem[MAX_ITEMS];  
  
        addItem("Vegetarian BLT",  
                "(Fakin') Bacon with lettuce & tomato on whole wheat", true, 2.99);  
        addItem("BLT",  
                "Bacon with lettuce & tomato on whole wheat", false, 2.99);  
        addItem("Soup of the day",  
                "Soup of the day, with a side of potato salad", false, 3.29);  
        addItem("Hotdog",  
                "A hot dog, with sauerkraut, relish, onions, topped with cheese",  
                false, 3.05);  
        addItem("Steamed Veggies and Brown Rice",  
                "Steamed vegetables over brown rice", true, 3.99);  
        addItem("Pasta",  
                "Spaghetti with Marinara Sauce, and a slice of sourdough bread",  
                true, 3.89);  
    }  
  
    public void addItem(String name, String description,  
                        boolean vegetarian, double price)  
    {  
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
          
        if (numberOfItems >= MAX_ITEMS) {  
            System.err.println("Sorry, menu is full!  Can't add item to menu");  
        } 
        else {  
            menuItems[numberOfItems] = menuItem;  
            numberOfItems = numberOfItems + 1;  
        }  
    }  
  
    public Iterator createIterator() {  
        return new DinerMenuIterator(menuItems);  
    }  
}
```

그리고, 이제 이 메뉴들을 차례대로 순환시킬 `DinerMenuIterator`클래스를 만듭니다. 해당 클래스는 `Iterator` 인터페이스를 상속받아 구현 합니다.

🗅 **<span style="color: #c03a92">class DinerMenu</span>**

```java
package src.iterator.dinerMerger;  
  
public class DinerMenuIterator implements Iterator {  
  
    private MenuItem[] items = null;  
    private int position = 0;  
  
    public DinerMenuIterator(MenuItem[] _items) {  
        this.items = _items;  
    }  
  
    @Override  
    public boolean hasNext() {  
        return items.length > position;  
    }  
  
    @Override  
    public MenuItem next() {  
        return items[position++];  
    }  
}
```

위의 과정과 마찬가지로, `PancakeHouseMenu` ,`PancakeHouseMenuIterator` 를 만들어 줍니다. 차이점은 요소들을 배열에 저장하는 것이 아닌 리스트에 저장한다는 것입니다.

🗅 **<span style="color: #c03a92">class PancakeHouseMenu</span>**

```java
package src.iterator.dinerMerger;  
  
  
import java.util.ArrayList;  
import java.util.List;  
  
public class PancakeHouseMenu implements Menu {  
    List<MenuItem> menuItems;  
  
    public PancakeHouseMenu() {  
        menuItems = new ArrayList<MenuItem>();  
  
        addItem("K&B's Pancake Breakfast",  
                "Pancakes with scrambled eggs and toast",  
                true,  
                2.99);  
  
        addItem("Regular Pancake Breakfast",  
                "Pancakes with fried eggs, sausage",  
                false,  
                2.99);  
  
        addItem("Blueberry Pancakes",  
                "Pancakes made with fresh blueberries",  
                true,  
                3.49);  
  
        addItem("Waffles",  
                "Waffles with your choice of blueberries or strawberries",  
                true,  
                3.59);  
    }  
  
    public void addItem(String name, String description,  
                        boolean vegetarian, double price)  
    {  
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);  
        menuItems.add(menuItem);  
    }  
  
    public List<MenuItem> getMenuItems() {  
        return menuItems;  
    }  
  
    public Iterator createIterator() {  
        return new PancakeHouseMenuIterator(menuItems);  
    }  
  
    public String toString() {  
        return "Objectville Pancake House Menu";  
    }  
}
```

🗅 **<span style="color: #c03a92">class PancakeHouseMenuIterator</span>**

```java
package src.iterator.dinerMerger;  
  
import java.util.List;  
  
public class PancakeHouseMenuIterator implements Iterator {  
    List<MenuItem> items;  
    int position = 0;  
  
    public PancakeHouseMenuIterator(List<MenuItem> items) {  
        this.items = items;  
    }  
  
    public MenuItem next() {  
        return items.get(position++);  
    }  
  
    public boolean hasNext() {  
        return items.size() > position;  
    }  
}
```

### 각 메뉴들을 읊을 웨이터 클래스 만들기

자 이제, 각 메뉴들이 담긴 클래스와 그 메뉴들을 하나씩 순회하는 기능까지 만들었습니다. 그 다음에는 `Iterator`를 호출하여 실행시킬 웨이터 클래스를 만듭니다.

해당 클래스 에서는 `MenuItem` 들을 순회하며 `MenuItem` 의 정보를 출력할 `printMenu` 라는 함수를 정의 합니다.

🗅 **<span style="color: #c03a92">class PancakeHouseMenuIterator</span>**

```java
package src.iterator.dinerMerger;  
  
public class Waitress {  
    Menu pancakeHouseMenu;  
    Menu dinerMenu;  
  
    public Waitress(Menu _pancakeHouseMenu, Menu _dinerMenu) {  
        this.pancakeHouseMenu = _pancakeHouseMenu;  
        this.dinerMenu = _dinerMenu;  
    }  
  
    public void printMenu() {  
        Iterator pancakeIterator = pancakeHouseMenu.createIterator();  
        Iterator dinerIterator = dinerMenu.createIterator();  
  
        System.out.println("메뉴\n----\n아침");  
        printMenu(pancakeIterator);  
        System.out.println("\n점심");  
        printMenu(dinerIterator);  
  
    }  
  
    private void printMenu(Iterator iterator) {  
        while (iterator.hasNext()) {  
            MenuItem menuItem = iterator.next();  
            System.out.print(menuItem.getName() + ", ");  
            System.out.print(menuItem.getPrice() + " -- ");  
            System.out.println(menuItem.getDescription());  
        }  
    }
}
```

`printMenu` 함수를 보시면, Iterator 를 받아서 while 문을 돌립니다. `Iterator` 클래스에는 boolean 값을 반환하는 hasNext() 라는 함수가 있었습니다. 다음 요소가 있다면 true, 없다면 false 를 반환하는 메서드 입니다. 이것이 false, 즉 다음 요소가 없을때까지 순서대로 순회한다고 생각하시면 됩니다.

또한 `next()`라는 함수로 다음 요소의 정보를 가져 옵니다.

### Main 함수에서 실행

```java
package src.iterator.dinerMerger;  
  
public class Main {  
    public static void main(String[] args) {  
    
        PancakeHouseMenu pancakeHouseMenu = new PancakeHouseMenu();  
        DinerMenu dinerMenu = new DinerMenu();  
        
        Waitress waitress = new Waitress(pancakeHouseMenu,dinerMenu);  
  
        waitress.printMenu();  
    }  
}
```

Menu 들이 담긴 클래스를 생성해주고, Waiteress 의 생성자에 끼워준 다음, `printMenu()` 를 호출하면 다음과 같이 각 요소들을 돌면서 메뉴 정보들을 출력하는 것을 볼 수 있습니다.

![](/images/Pasted%20image%2020240916122405.png)


## 마무리

Iterator 패턴은 보통 컬렉션들을 순회하는데에 많이 사용 됩니다. C# 에서도 List 를 보면 IEnumerable<T> 라는 클래스를 상속받고 있고, IEnumerable<T> 클래스 안을 살펴보면 IEnumerator 라는 인터페이스가 안에 들어가 있는 것을 볼 수 있습니다. 

또한, IEnumerable<T>를 구현한 클래스는 모두 foreach 에 들어갈 수 있습니다.

![](/images/Pasted%20image%2020240916122936.png)

![](/images/Pasted%20image%2020240916123009.png)

지금은 컬렉션을 구현할 때, 언어 차원에서 Iterator 패턴을 구현하여 제공하기 때문에, 많이 사용할 일을 없을 것 같습니다. 하지만 컬렉션이 어떻게 구현되어 있는지를 볼 수 있어서 뜻깊은 시간이었던것 같습니다.