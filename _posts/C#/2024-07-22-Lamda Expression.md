---
title:  "[C#] 람다식 이란?" 

categories:
  -  C Sharp
tags:
  - [Programming, Language]

toc: true
toc_sticky: true

published: true

date: 2020-07-22
last_modified_at: 2020-07-22
---

## 람다식이란?

이제는 거의 모든 언어에서 사용되고 있는 람다식 입니다. 람다식이란 이름과 형태가 어렵지만, 간단하게말해서 `익명함수`라고 생각하면 됩니다.

함수는 원래 `접근제한자, 반환형, 이름, 인자, 구현부` 이렇게 구현되어 있습니다. 여기에서 `접근제한자, 이름`을 제외 한 것이  바로 람다식 입니다.

사용방법은 다음과 같습니다. 이름이 없는 반환형과 매개변수가 모두 `void`인 람다식 입니다.

```csharp
() => {Console.WriteLine("구현부 입니다.");}
```

다음 예제는 C# 책에 있는 예제를 가져왔습니다.

## 예제

### 람다식 모양

```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        private delegate int Calculate(int a, int b);
        static void Main(string[] args)
        {
            Calculate cal = (int a, int b) => a + b;
            Calculate cal = ( a,  b) => a + b;
            Calculate cal = delegate (int a, int b)
		    {
			  return a + b;
		    };       
        }
    }
}

```

요 3가지 형태로 쓸 수 있음
델리게이트로 사용하는게 원래 C# 2.0에 있었는데 C# 3.0에 람다식이 추가되어서 둘다 공존
#### 실제 사용
```C#
namespace LamdaTest
{
    internal class Program
    {
        private delegate int Calculate(int a, int b);
        static void Main(string[] args)
        {
            Calculate calc = (a, b) => a + b;
            Console.WriteLine(calc(4,3));

        }
    }
}
```


## 문 형식

- 델리게이트 안에서 특정 로직을 처리할 수 있는 형식
```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        private delegate void DoSomething();
        static void Main(string[] args)
        {
            DoSomething DoIt = () =>
            {
                Console.WriteLine("뭔가를");
                Console.WriteLine("출력해보자");
                Console.WriteLine("이런식으로");
            };
            DoIt();

        }
    }
}

```

```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        private delegate string Concatenate(string[] args);
         static void Main(string[] args)
        {
            Concatenate concat = (arr) =>
            {
                string result = "";
                foreach (string s in arr)
                {
                    result += s;
                }
                return result;
            };

            Console.WriteLine(concat(args));
        }
    }
}

```


## Func와 Action으로 간결하게 무명함수 만들기

- 익명 메소드와 무명함수는 코드를 더 간결하게 해줌 
- 매번 별개의 대리자가 생성되야 함
- 그래서 C#은 Func와 Action 대리자를 미리 생성해놨음
- `Func`은 결과를 반환하는 녀석, `Action`은 결과를 반환하지 않는 녀석


### Func
- 총 17가지 버전의 Func이 있음
- 매개변수 중 가장 마지막에 있는 것이 반환 형식임

#### 매개변수가 없는 버전
```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Func<int> func1 = () => 10;
            Console.WriteLine(func1());
        }
    }
}

```

#### 매개변수가 1개인 버전

```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Func<int, int> func2 = (x) => x * 2;
            Console.WriteLine(func2(9));
        }
    }
}

```

#### 매개변수가 2개인 버전

```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Func<int, int, int> func3 = (x, y) => (x + 2) * (y + 3);
            Console.WriteLine(func3(1,2));
        }
    }
}

```

### Action

- Action 대리자는 Func 대리자와 거의 똑같음
- 차이점은 Action 대리자는 반환 형식이 없음
- 이것도 17개의 버전이 있음

#### 매개변수 없는 버전
```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            Action act1 = () => Console.WriteLine("Action()");
            act1();
        }

    }
}

```

#### 매개변수가 하나뿐인 버전
```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            int result = 0;
            Action<int> action = (x) => result = x * x;
            Console.WriteLine(3);
        }

    }
}

```

### 매개변수가 2개인 버전

```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LamdaTest
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            int result = 0;
            Action<int, int> action = (x, y) =>
            {
                result = x + y;
                Console.WriteLine(result);
            };
            action(2, 3);
        }

    }
}

```