---
title: "[Clean Architecture] Clean Architecture DIP(Dependency Inversion Principle 의존성 역전의 원칙ㅔ)"

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

## 의존성 역전의 원칙 DIP

의존성 역전 원칙에서 말하는 **유연성이 극대화된 시스템**이란 소스코드 의존성이 추상에 의존하며 구체에는 의존하지 않는 시스템 입니다.

우리가 의존하지 않도록 피하고자 하는 것은 바로 변동성이 큰 구체적인 요소입니다. 그리고 이  구체적인 요소는 우리가 열심히 개발하는 중이라 자주 변경될 수밖에 없는 모듈들 입니다.

추상 인터페이스에서 변경이 생기면 이를 구체화한 구현체들도 따라서 수정해야 합니다. 반대로 구체적인 구현체에 변경이 생기더라도 그 구현체가 구현하는 인터페이스는 항상, 좀 더 정확히 말하면 대다수의 경우 변경될 필요가 없습니다. 따라서 인터페이스는 구현체보다 변동성이 낮습니다.

실제로 뛰어난 소프트웨어 설계자와 아키텍트라면 인터페이스의 변동성을 낮추기 위해 애를 씁니다. 인터페이스를 변경하지 않고도 구현체에 기능을 추가할 수 있는 방법을 찾기 위해 노력합니다. 이는 소프트웨어 설계의 기본 입니다. 즉, 안정된 소프트웨어 아키텍처란 변동성이 큰 구현체에 의존하는 일은 지양하고, 안정된 추상 인터페이스를 선호하는 아키텍처라는 뜻입니다.

이 원칙에서 전달하려는 내용은 다음과 같이 매우 구체적인 코딩 실천법으로 요약할 수 있습니다.

- 변동성이 큰 구체 클래스를 참조하지 말라. 대신 추상 인터페이스를 참조하라. 이 규칙은 언어가 정적 타입이든 동적 타입이든 관계없이 모두 적용된다. 또한 이 규칙은 객체 생성방식을 강하게 제약하며, 일반적으로 추상 팩토리를 사용하게 만든다.
- 변동성이 큰 구체 클래스로부터 파생하지 말라. 이 규칙은 이전 구칙의 따름정리이지만, 별도로 언급할 만한 가치가 있다. 정적 타입 언어에서 상속은 소스코드에 존재하는 모든 관계 중에서 가장 강력한 동시에 뻣뻣해서 변경하기 어렵다. 따라서 상속은 아주 신중하게 사용해야 한다.
- 구체 함수를 오버라이드 하지 말라. 대체로 구체 함수는 소스 코드 의존성을 필요로 한다. 따라서 구체 함수를 오버라이드 하면 이러한 의존성을 제거할 수 없게 되며, 실제로는 그 의존성을 상속하게 된다. 이러한 의존성을제거하려면 차라리 추상 함수로 선언하고 구현체들에서 각자의 용도에 맞게 구현해야 한다.
- 구체적이며 변동성이 크다면 절대로 그 이름을 언급하지 마라. 사실 이 실천법은 DIP 원칙을 다른 방식으로 풀어 쓴것이다.

## DIP 위반 사례

우선 DIP 위반 사례를 하나 살펴봅니다. Client에서 OS를 판별한 후에, 라디오 버튼을 랜더링 한다고 가정합니다. 

```cs
namespace DIP.BadExample
{
    public class MacRadioButton
    {
        public void Render()
        {
            //..
        }
    }
}
```

```cs
namespace DIP.BadExample
{
    public class WindowRadioButton
    {
        public void Render()
        {
            //..
        }
    }
}
```

```cs
namespace DIP.BadExample
{
    public class Client
    {
        private MacRadioButton macRadioButton = null;
        private WindowRadioButton windowRadioButton = null;

        private string currentOs = null;

        public Client()
        {
            macRadioButton = new MacRadioButton();
            windowRadioButton = new WindowRadioButton();

            currentOs = "Mac";
        }

        public void Main()
        {
            if (currentOs == "Mac")
            {
                macRadioButton.Render();
            }
            else
            {
                windowRadioButton.Render();
            }
        }
    }
}
```

해당 코드 구조는 Client -> MacRadioButton / Client -> WindowRadioButton 으로 의존하고 있습니다. 이렇게되면 MacRadioButton 또는 WindowRadioButton에 Client가 의존하고 있으므로 변경에 취약해 집니다.

해당 예제를 IRadioButton을 추가하여 변경하여 봅니다.


```cs
public interface IRadioButton
{
    public void Render();
}
```

```cs
namespace DIP.GoodExample
{
    public class MacRadioButton : IRadioButton
    {
        public void Render()
        {
            //..
        }
    }
}
```

```cs
namespace DIP.GoodExample
{
    public class WindowRadioButton : IRadioButton
    {
        public void Render()
        {
            //..
        }
    }
}
```

```cs
namespace DIP.GoodExample
{
    public class Client
    {
        public IRadioButton radioButton = null;
        private string currentOs = "Window";

        public Client()
        {
            if (currentOs == "Window")
            {
                radioButton = new WindowRadioButton();
            }
            else
            {
                radioButton = new MacRadioButton();
            }
        }

        public void Main()
        {
            radioButton.Render();
        }

    }
}
```

위의 예시처럼 IRadioButton 이라는 인터페이스를 만들어서 Client -> IRadioButton / WindowRadioButton -> IRadioButton / MacRadioButton -> IRadioButton 으로 의존관계가 구성되었습니다. 이제는 Client 가 구체적인 RadioButton 클래스를 의존하지 않고, 추상적인 인터페이스에 의존하면서 의존관계가 역전된것을 볼 수 있습니다.

지금은 Window/Mac 운영체제를 자동으로 구분할 방법이 없어 if문으로 구분하지만, 자동으로 OS의 종류를 알 수 있게 된다면 Client의 수정없이 OS가 추가될때마다 확장이 가능합니다. 또한, Mac/Window RadioButton의 구현이 바뀐다고해도 Client에는 영향이 없게 됩니다.

## 결론

DIP는 객체지향 프로그래밍에서만 구현 가능한 원칙으로, 객체지향의 강력한 확장성을 실현하는 데 큰 역할을 합니다. 물론 모든 클래스가 구체적인 구현 대신 추상 인터페이스에 의존하도록 만드는 것이 항상 필요하거나 가능하지 않을 수 있습니다. 하지만 확장이나 잦은 변경이 예상된다면, 추상화에 의존하도록 설계하여 소프트웨어의 유연성을 높이는 것이 바람직합니다.