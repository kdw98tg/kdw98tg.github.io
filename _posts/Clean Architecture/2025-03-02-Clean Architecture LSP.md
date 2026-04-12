---
title: "[Clean Architecture] Clean Architecture LSP(Liskov Substitution Principle 라스코프 치환 원칙)"

categories:
  -  Clean Architecture
  
tags:
  - [Clean Architecture]

toc: true
toc_sticky: true

published: true

date: 2025-03-02
last_modified_at: 2025-03-02
---

## 라스코프 치환 원칙 (LSP)

1988년 바바라 라스코프는 하위 타입을 아래와 같이 정의하였습니다.

> 여기에서 필요한 것은 다음과 같은 치환 원칙이다. S타입의 객체 o1 각각에 대응하는 T 타입 객체 o2가 있고, T 타입을 이용해서 정의한 모든 프로그램 P에서 o2의 자리에 o1을 치환하더라도 P의 행위가 변하지 않는다면 S는 T의 하위 타입이다.

즉, 간단하게 말하면 자식타입은 언제나 부모타입을 교체할 수 있어야 한다는 것입니다.

리스코프 치환 원칙은 이름만 봐서는 어떤 원칙인지 상상하기 어렵습니다. 그래서 대표적인 라스코프 치환 원칙을 위반한 예제를 살펴봅시다. 

## 라스코프 치환 원칙 위반 예제 (정사각형/직사각형 문제)

우선, 정사각형과 직사각형의 특성에 대해 생각해 봅시다. 직사각형은 높이와 너비가 서로 다를 수 있지만, 정사각형은 높이와 너비가 서로 같은 사각형을 말합니다. 그래서 직사각형이 정사각형의 상위 개념이라고 생각할 여지가 있습니다.

코드로 간단한 예시를 만들면 다음과 같습니다.

```cs
public class Rectangle
{
    protected int height = 0;
    protected int width = 0;
    public virtual void SetHeight(int _height)
    {
        //높이를 설정하는 메서드
        this.height = _height;
    }

    public virtual void SetWidth(int _width)
    {
        //너비를 설정하는 메서드
        this.width = _width;
    }

    public virtual int GetArea()
    {
        return height * width;
    }
}
```

```cs
public class Square : Rectangle
{
    public override void SetHeight(int _height)
    {
        base.SetHeight(_height);
        base.SetWidth(_height);
    }

    public override void SetWidth(int _width)
    {
        base.SetWidth(_width);
        base.SetHeight(_width);
    }
}
```

클래스만 봤을때는, 별로 문제가 없어보입니다. 하지만 이런 클래스를 만들어놓고 테스트케이스로 다음과 같은 Assert를 넣었다고 생각해 봅시다.

```cs
using System.Diagnostics;
using System.Reflection;

public class User
{
    public void Main()
    {
        Rectangle rectangle = new Rectangle();
        rectangle.SetHeight(5);
        rectangle.SetWidth(2);
        Debug.Assert(rectangle.GetArea() == 10);
    }
}
```

부모클래스인 직사각형의 경우를 생각하면 해당 Assert는 가볍게 통과할 것입니다. 하지만, 다형성을 이용해서 자식 클래스인 Square를 생성한다면 어떻게 될까요?

```cs
using System.Diagnostics;
using System.Reflection;

public class User
{
    public void Main()
    {
        Rectangle square = new Square();
        square.SetHeight(5);
        square.SetWidth(2);
        Debug.Assert(square.GetArea() == 10);
    }

}
```

이렇게 한다면, Assert를 통과하지 못할 것입니다. 왜나하면 정사각형의 높이와 너비는 같기 때문에, 최종 정사각형의 높이와 너비는 2 가 되는 것입니다. 그래서 예상했던 10이 나오는것이 아닌 4 라는 값이 나와서 Assert에 통과하지 못하는 것입니다.

결국, 다형성을 이용하려면 클라이언트의 코드를 바꾸거나, 조건으로 타입을 구분해줘야 하는데 이는 좋은 구조가 아닙니다. 결국 클라이언트에서 호출할 때, 조건이나 타입에 의존하게 되므로, **타입을 서로 치환할 수 없다** 가 됩니다.


## LSP 와 아키텍쳐

객체지향이 혁명처럼 등장한 초창기에는 앞서 본 것처럼 LSP는 상속을 사용 하도록 가이드하는 방법 정도로 간주되었습니다. 하지만 시간이 지나면서 LSP는 인터페이스와 구현체에도 적용되는 더 광범위한 소프트웨어 설계 원칙으로 변모해 왔습니다.

아키텍쳐 관점에서 LSP를 이해하는 최선의 방법은 이 원칙을 어겼을 때, 시스템 아키텍쳐에 무슨일이 일어나는지 관찰하는 것입니다.

만약 LSP를 지키지 않고 설계를 한다면, 확장을 할 때마다 클라이언트 코드에 분기문을 넣던지, 타입을 구분시켜 주던지의 추가 작업이 필요합니다. 이렇게해도 소프트웨어는 동작하겠지만, 확장에 있어서 분명히 제한이 생기고 해당 코드를 사용하는 클라이언트의 입장에서도 알아야 할 정보가 너무 많기 때문에 오래가지 못해 그 소프트웨어는 유지보수가 불가능하게 될 것입니다.

## 결론
LSP 는 결국 다형성의 특징을 이용하기 위해 상위 클래스 타입으로 객체를 선언하여 하위 클래스의 인스턴스를 받으면, 업캐스팅된 상태에서 부모의 메서드를 사용해도 동작이 의도대로 흘러가도록 하는 원칙 입니다.

하지만 주의할 점은, 객체지향 프로그래밍에서 상속은 기반 클래스와 서브 클래스 사이에 IS-A 관계가 있을 때만으로 제한되어야 한다는 점입니다. (그 외의 경우에서는 Composition을 이용하도록 권고되어 있음)

따라서 다형성을 이용하고 싶다면 extends 대신에 인터페이스로 사용하기를 권하며, 상위 클래스의 기능을 이용하거나 재사용을 하고 싶다면 상속 보단 합성으로 구성하기를 권장합니다.