---
title: "[Clean Architecture] 경계 해부학 및 업무 규칙"

categories:
  -  Clean Architecture
  
tags:
  - [Clean Architecture]

toc: true
toc_sticky: true

published: true

date: 2025-05-16
last_modified_at: 2025-05-16
---

## 경계 해부학

저번 포스팅에서는 경계를 잘 나눠서 관리해야 한다는 이야기를 했습니다. 그렇다면 그 경계를 어떻게 나눠야 하는걸까요? 일단, 경계를 잘 나누는것은 `소스코드의 의존성을 잘 관리한다`라는 말과 일맥상통합니다. 즉, 경계는 모듈간의 의존성을 잘 관리해서 업무규칙이 바뀌었을때, 최소한의 노력으로 소프트웨어를 변경시킬 수 있도록 해줍니다. (저자는 이를 `방화벽`이라고 표현합니다.)

경계를 나눌 수 있게 된 배경에는 `객체지향`이라는 소프트웨어 패러다임이 있었기 때문입니다. 객체지향이 없었다면, 또는 다형성에 해당하는 메커니즘이 없었다면, 아키텍트는 결합도를 적절히 분리하기 위해 함수를 가리키는 포인터라는 위험한 옛 관행에 기대야만 했을것입니다.

예를들어 다음과 같은 구조가 있다고  해봅니다.

![](/images/Pasted%20image%2020250516154400.png)

해당 구조를 보면, 고수준인 Client가 상대적으로 저수준인 Service와 Data에 의존하고 있는것을 볼 수 있습니다. 이것은 경계를 잘 나눴다고 볼 수 없습니다. 

상식적으로 생각한다면, Client는 Service와 Data에 참조를 가지고 Service와 Data를 조작하는 파사드(Facade)라고 생각할 수 있습니다. 그럼 저수준이 고수준 모듈에 의존하도록 바꾸려면 어떻게 해야할까요? 바로 객체지향의 꽃인 `DIP`를 통해서 해결할 수 있습니다.

고수준인 Client 모듈쪽에 인터페이스를 만들고, 저수준의 모듈이 고수준의 모듈인 Interface에 의존하도록 만드는것입니다. 다음 그림과 같습니다.

![](/images/Pasted%20image%2020250516154649.png)

인터페이스를 두고, 경계를 그었습니다. 저수준인 Service 구현체는 이제 고수준인 Service 인터페이스에 의존하게 되었습니다. 이렇게 하였을때, 장점은 업무 정책이 바뀌더라도 고수준의 모듈에서 변화가 거의 생기지 않고, 저수준의 모듈만 새로 정의한 후, 붙여주면 된다는 것입니다. 

또한, 팀작업을 하다는 기준으로 보게 되면, 팀들은 서로의 영역에 침범하지 않은 채 자신만의 컴포넌트를 독립적으로 작업할 수 있다는 이점이 있습니다. 인터페이스에 맞게 작업한 결과물을 갈아끼우기만 하면 되기 때문입니다.

코드로 다른 예를 들어보겠습니다. 

다음은 `Translate` 모듈이 의존하고 있는 `CharWriter`, `CharReader`를 소스코드로 표현한 것입니다. 다음 소스코드는 고수준의 모듈이 저수준의 모듈에 의존하고 있고, 아키텍쳐 관점에서 잘못된 예입니다.

```cs
namespace DependencyDirection.Incorrect
{
    //의존성의 흐름이 고수준 -> 저수준 모듈로 향한다
    //Client -> Translate |-> CharReader 
    //                    |-> CharWriter
    //좋은 방향은 저수준 모듈에서 고주순 모듈로 향하게 하는것이다.
    //인터페이스로 묶어서 사용하도록 한다.
    public class Client
    {
        private readonly Translate translate;

        public Client()
        {
            translate = new Translate();
        }

        public void Main()
        {
            translate.TranslateText("Some Text");
        }
    }
}
```

```cs
namespace DependencyDirection.Incorrect
{
    public class Translate
    {
        private readonly CharWriter writer;
        private readonly CharReader reader;

        public Translate()
        {
            writer = new CharWriter();
            reader = new CharReader();
        }

        public string TranslateText(string _text)
        {
            string text = reader.Read(_text);
            return writer.Write(text);
        }
    }
}
```

```cs
namespace DependencyDirection.Incorrect
{
    public class CharWriter
    {
        public string Write(string _text){
            //some logic
            return _text;
        }
    }
}
```

```cs
namespace DependencyDirection.Incorrect
{
    public class CharReader
    {
        public string Read(string _text){
            //some logic
            return _text;
        }
    }
}
```

지금 소스코드의 의존성 흐름을 보면, Client -> Translate -> CharReader, CharWriter 로 고수준의 모듈이 저수준의 모듈에 의존하고 있는것을 볼 수 있습니다. 만약에 CharReader의 소스코드가 바뀌면, Translate, Client 모듈에게 영향이 가는 구조입니다.

![](/images/Pasted%20image%2020250516155517.png)

그렇다면, 이것을 바꿔서 저수준의 모듈이 고수준의 모듈에 의존성이 있도록 바꿀 수 있을까요? 위에서 말하였듯 바로 `DIP`를 활용하는 것입니다.

Client, Translate, ICharReader, ICharWriter를 고수준의 모듈로 묶고, 저수준의 구현체와 선을 긋는 것입니다.

이를 그림으로 표현하면 다음과 같습니다.

![](/images/Pasted%20image%2020250516155935.png)

그리고, 이를 소스코드로 표현하면 다음과 같습니다.

```cs
namespace DependencyDirection.Correct
{
    //저수준의 모듈 (입출력 관련) 에서 고수준 모듈로 의존성을 역전하였음
    //이렇게하면 고수준의 모듈은 저수준의 모듈에게 의존성이 없어짐
    //이게 옳게된 구조라고 함
    //지금 나눈 Client 와 LowLevelModeul이 경계선이 되는것임
    public class Client
    {
        private readonly Translate translate;

        public Client()
        {
            translate = new Translate();
        }

        public void Main()
        {
            translate.Translate("Some Text");
        }
    }
}
```

```cs
namespace DependencyDirection.Correct
{
    public interface ICharReader
    {
        public string Read(string _text);
    }
}
```

```cs
namespace DependencyDirection.Correct
{
    public interface ICharWriter
    {
        public string Write(string _text);
    }
}
```

```cs
namespace DependencyDirection.Correct
{
    public class Translate
    {
        private readonly ICharWriter writer;
        private readonly ICharReader reader;

        public Translate()
        {
            writer = new CharWriter();
            reader = new CharReader();
        }

        public string Translate(string _text)
        {
            string text = reader.Read(_text);
            return writer.Write(text);
        }
    }
}
```

```cs
namespace DependencyDirection.Correct
{
    public class CharReader : ICharReader
    {
        public string Read(string _text)
        {
            //some logic
            return _text;
        }
    }
}
```

```cs
namespace DependencyDirection.Correct
{
    public class CharWriter : ICharWriter
    {
        public string Write(string _text)
        {
            //some logic
            return _text;
        }
    }
}
```

이제, 읽고 쓰는 모듈이 ByteReader가 됐든, ConsoleReader가 됐든, StringReader가 됐든 뭐가 와도 고수준의 모듈에는 변화가 가의 일어나지 않게 됩니다.

따라서 이렇게 정의할 수 있습니다.

> 저수준의 서비스는 반드시 고수준의 서비스에 `플러그인` 되어야 한다.
> 고수준 서비스의 소스 코드에는 저수준 서비스를 특정 짓는 어떤 물리적인 정보도 절대 포함해서는 안된다.

보통 입출력 장치 또는 데이터베이스에 접근하는 모듈들은 저수준의 모듈일 가능성이 크므로, 경계를 나누는 대상이 되어야 합니다.

## 정책과 수준

서비스의 정책 즉, 업무규칙은 사업적으로 수익을 얻거나 비용을 줄일 수 있는 규칙 또는 절차입니다. 더 엄밀하게 말하면 컴퓨터상으로 구현했는지와 상관없이, 업무규칙은 사업적으로 수익을 얻거나 비용을 줄일 수 있어야 합니다.

즉, 컴퓨터로 계산하든, 주판을 두두려서 계산을 하든 그것은 업무규칙과는 무관한 영역(저수준)이라는 것입니다. 

이것을 추상화하여 Entity라고 부르겠습니다. Entity는 컴퓨터 시스템 내부의 객체로서, 핵심 업무 데이터를 기반으로 동작하는 일련의 조그만 핵심 업무 규칙을 구체화합니다. 대출 엔티티를 예로 들어보겠습니다.

대출엔티티는 `principle`, `rate`, `period` 등등의 데이터를 포함하고 있습니다. (핵심 업무 규칙은 보통 데이터를 요구함) 대출 엔티티, 더 넓게는 엔티티는 업무규칙에 대한 데이터를 포함하는 역할만 해야지, 데이터베이스, 사용자 인터페이스, 서드파티 프레임워크에 대한 고려사항들로 오염되면 안됩니다.

업무규칙은 소프트웨어 시스템이 존재하는 이유이고, 업무 규칙은 핵심적인 기능입니다. 이상적으로는 업무 규칙을 표현하는 코드는 반드시 시스템의 심장부에 위치해야 하며, 덜 중요한 코드는 이 심장부에 플러그인 되어야 합니다. 업무 규칙은 시스템에서 가장 독립적이며 가장 많이 재사용할 수 있는 코드여야 합니다.

가장 많이 재사용할 수 있는 코드여야 하는 예는 다음과 같습니다. 이 유저가 성인인지 아닌지를 판단하는 로직입니다. Entity를 하나 만들고, 거기에 IsAdult()라는 함수를 만들고 Client에서 재사용 합니다. 이렇게 되면, 성인으로 인정하는 기준이 나라마다 달라도, UserEntity만 수정하거나 교체하면 모든 참조하고 있는 곳에서 반영이 됩니다.

```cs
namespace EntityRecycle.Correct
{
    public class UserEntity
    {
        public string Name { get; private set; }
        public int Age { get; private set; }

        public bool IsAdult()
        {
            return Age >= 19;
        }
    }
}
```

```cs
namespace EntityRecycle.Correct
{
    public class Client
    {
        private readonly UserEntity user;

        public Client()
        {
            user = new UserEntity();
        }

        public void Main()
        {
            if (user.IsAdult())
            {
                //SomeLogic
            }
        }

        public void UpdateAdult()
        {
            if (user.IsAdult())
            {
                //Some Login
            }
        }
    }
}
```

반면, Entity에서 캡슐화된 함수가 없이, Client에서 반복되는 코드를 작성하면, 관리포인트가 늘어나며, 버그가 많아질 가능성이 급증하게 됩니다.

```cs
namespace EntityRecycle.Incorrect
{
    public class Client
    {
        private readonly UserEntity user;

        public Client()
        {
            user = new UserEntity();
        }

        public void Main()
        {
            if (user.Age > 19)
            {
                //SomeLogic
            }
        }

        public void UpdateAdult()
        {
            if (user.Age > 19)
            {
                //Some Login
            }
        }
    }
}
```

업무규칙은 소프트웨어 시스템이 존재하는 이유입니다. 따라서, 업무규칙을 독립시키고, 원래 그대로의 모습으로 남아있도록 만드는것이 중요합니다.