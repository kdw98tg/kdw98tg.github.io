---
title: "[Unity] Coroutine 이란? Coroutine 의 활용법"

categories:
  -  Unity
  
tags:
  - [Unity, C#]

toc: true
toc_sticky: true

published: true

date: 2024-03-19
last_modified_at: 2024-03-19
---

## 코루틴 이란?

유니티 기본 문서에서는 `코루틴(Coroutine)`을 다음과 같이 정의하고 있습니다.

>대부분의 경우 메서드를 호출하면 실행을 완료한 뒤 호출한 메서드에 제어와 선택적 반환 값을 반환합니다. 즉, 메서드 내에서 발생한 모든 행동은 단일 프레임 업데이트에서 발생해야 합니다.

>반면, 코루틴을 사용하면, 작업을 다수의 프레임에 분산할 수 있습니다. 유니티에서 코루틴은 실행을 일시 정지하고, 제어를 유니티에 반환하지만, 중단한 부분에서 다음 프레임을 계속할 수 있는 메서드 입니다.

간단하게 말하면, 시간의 경과에따른 명령을 주고싶을 때 사용하게 되는 문법 입니다. 

처음에는 코루틴은 멀티스레딩을 하게 하는 메서드 인줄 알고 있습니다.

하지만, 코루틴은 실제로 멀티스레딩 처리를 하는 것이 아닌 하나의 스레드(싱글 스레드)에서 비동기처럼 작동하는 메서드 입니다.

유니티 공식 문서에서도 위와 같은 내용을 포함하고 있습니다.

> 코루틴은 스레드가 아니라는 점을 명심해야 합니다.
> 
> 코루틴의 동기 작업은 여전히 메인 스레드에서 실행됩니다.
> 
> 메인 스레드에 소요되는 CPU 시간을 줄이려면 다른 스크립트 코드에서와 마찬가지로코루틴의 작업 차단을 방지하는 것이 중요합니다.
> 
> Unity 내에서 다중 스레드 코드를 사용하려면, C# 잡 시스템을 고려하세요. 

### 코루틴의 실체

**코루틴은 싱글스레드에서 동작하는 비동기 작업방식의 패러다임 입니다.** 
즉, 코루틴은 멀티스레딩이 아니기 때문에 운영체제의 스케쥴링과 전혀 연관이 없습니다. 또한, 싱글스레드이기 때문에, 멀티 스레딩으로 인한 동기화 이슈 역시 전혀 발생하지 않습니다.  

간략하게, `IEnumerator`를 반환하는 메서드에 yield 예약어를 사용하면 컴파일러는 컴파일 시점에 이를 상태머신(State Machine)으로 변환합니다.

코루틴의 진짜 정체는 **상태머신** 이었던 것입니다.

상태머신은 메서드가 어디서 중단되었는지, 다음에 어디서부터 실행돼야 하는지를 추적합니다. 그리고, 유니티의 코루틴 시스템은 마치 운영체제처럼 이러한 코루틴들의 실행 흐름을 내부로직을 통해 조율 합니다.

즉, 코루틴은 매 프레임 호출되어 `IEnumerator`의 `MoveNext()`를 호출합니다.
이는 `yield return`까지의 코드가 실행됨을 의미합니다. 그리고, `yield return`으로 반환된 상태에 따라 코루틴의 상태가 설정됩니다. 즉, 코드의 흐름을 스케쥴링 하는 것입니다.

### 탄생 배경

기존의 .NET 프로그래밍에서는 `Thread`나 `Task` 등을사용하여 멀티스레딩으로 비동기 작업을 하는것이 관례였습니다. 
하지만 유니티 뿐만 아니라 게임엔진에서는 멀티스레딩 사용을 지양합니다.
왜냐하면 물리엔진이라는 것은 물리 현상의 다양한 상태와 복잡한 연산을 처리하는 프로그램인데, 멀티 스레딩으로 접근할경우 치명적인 오류가 발생할 수 있기 때문입니다.

그래서 싱글스레드에서 비동기 작업을 할 수 있도록 코루틴이라는 것을 도입하였습니다.

## 왜 코루틴을 사용하는가?

유니티에서 코루틴은 비동기 작업을 처리하면서도 게임 루프의 제어를 유지할 수 있기 때문에 사용합니다. 비동기 작업 이라고 하였지만, 위에서 설명하였던 대로, 사실은 싱글 스레드에서 일어나는 작업이며, 불필요하게 스레드를 늘릴 필요 없이 로직을 돌릴 수 있기 때문에 사용합니다.



## 코루틴의 특징

- 반드시 `IEnumerator`를 반환해야 함
- `yield return`을 만나는 순간마다 다음 구문이 실행되는 프레임으로 나뉘게 됨
- `yield break`를 만나면 코루틴 종료
- `MonoBehaviour`를 상속 받아야 함
- 코루틴에게는 `소유권`이라는 개념이 있는데, 소유권을 가진 객체가 비활성화 되거나 파괴되면 해당 객체가 소유한 모든 코루틴은 종료가 됨
- 코루틴은 메인스레드에서 실행됨. 절대 멀티 스레드가 아님

## 코루틴 사용법

코루틴을 사용하는 방법에는 여러가지가 있습니다.

예제를 통해서 살펴 보도록 합니다.

코루틴에는 `yield return` 이라는 구문이 꼭 들어가야 합니다. 

위에서 설명했듯이, 이 `yield return` 키워드를 통해서 많은 것 들을 할 수 있습니다.

### 1. 1프레임 체크하기

`yield return null` 을 사용하게 되면 한프레임 쉬고, 다음 로직을수행하게 됩니다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    private void Start()
    {
        print($"Start : {Time.deltaTime}");
        StartCoroutine(MyCo());
    }
    private IEnumerator MyCo()
    {
        print($"MyCo0 : {Time.time}");
        yield return null;
        print($"MyCol : {Time.time}");
    }
}
```

![실행결과](/images/Pasted%20image%2020240319152309.png)

이것을 잘 활용하면 코루틴을 Update() 처럼 사용할 수 있습니다.

### 2. Update() 처럼 사용하기

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    private float value05 = 5f;
    private void Start()
    {
        StartCoroutine(MyCo());
    }
    private IEnumerator MyCo()
    {
        while (true)
        {
            value05 += Time.deltaTime;
            if (value05 > 5f) break;

            yield return null;
        }
    }
}
```

`while`을 통해서 무한이 반복을 돌리는 코드 입니다. 이렇게 하면 Update()와 같은 효과를 낼 수 있습니다.

기본적으로, `Update()`는 매프레임 60번씩 호출되어 성능상 좋지는 않습니다. 그렇기 때문에, 코루틴에서 체크하다가, 코루틴이 끝나면 더이상 체크하지 않도록 하는것이 좋습니다.

반대로 이야기하면, 코루틴에서 무한반복되는 체킹과정을 거친다면, 그냥 업데이트를 사용하는것이 좋습니다.

즉, 끝날시점이 정해져있으면 코루틴을 사용하면 됩니다. 

### 3. 흐름제어 객체를 변수화하기 및 다양한 흐름제어

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    private void Start()
    {
        StartCoroutine(MyCo());
    }
    private IEnumerator MyCo()
    {
        yield return new WaitForSeconds(1f);
        print("1");
        yield return new WaitForSeconds(1f);
        print("1");
        yield return new WaitForSeconds(1f);
        print("1");
    }
}
```

흐름제어객체를 호출할 때 마다, 가비지가 생기므로, 변수로 등록하여 사용하는 것이 좋습니다.

아래는 변수화한 코드 입니다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    private readonly WaitForSeconds delayOneSecond = new WaitForSeconds(1);
    private void Start()
    {
        StartCoroutine(MyCo());
    }
    private IEnumerator MyCo()
    {
        yield return delayOneSecond;
        print("1");
        yield return delayOneSecond;
        print("1");
        yield return delayOneSecond;
        print("1");
    }
}
```


```cs
yield return new WaitForSecondsRealtime(1f);
```
`Time.timeScale = 0.5f` 해도 이거쓰면 안느려집니다.

```cs
yield return new WaitForEndOfFrame();
```
프레임 끝날 때 호출됩니다.


### 코루틴 체인 활용

코루틴을 활용하여 서로 연관되어 있는 코드들을 실행시킬 수 있습니다.
그것을 `코루틴 체인`이라고 하더라고요.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField]
    [Range(0f, 2f)]
    private float value02 = 0f;
    [SerializeField]
    [Range(0f, 2f)]
    private float value03 = 0f;

    private void Start()
    {
        StartCoroutine(MyCo());
    }
    private IEnumerator MyCo()
    {
        //코루틴을 반환하기 때문에 코루틴이 끝날 동안을 대기함

        yield return StartCoroutine(Value02Co());
        yield return StartCoroutine(Value03Co());
        print("모든 코루틴이 끝남");
    }
    private IEnumerator Value02Co()
    {
        while (true)
        {
            value02 += Time.deltaTime;
            if(value02 > 2)
            {
                value02 = 2f;
                break;
            }
            yield return null;
        }
    }
    private IEnumerator Value03Co()
    {
        while (true)
        {
            value03 += Time.deltaTime;
            if (value03 > 2)
            {
                value03 = 2f;
                break;
            }
            yield return null;
        }
    }
}
```

위와같이 코루틴을 서로 연결하여 `코루틴 체인`을 만들 수 있습니다.
`yield return StartCoroutine()`을 하면 코루틴을 실행하고, 해당 코루틴이 끝날 때 까지 기다리게 됩니다.

### 코루틴변수 할당

`StartCoroutine()`의 인자에 `IEnumerator` 형식을 넣게 되어있습니다. 그런데, 여기서 `IEnumerator`를 반환하는 함수를 넣었을 때, 같은 코루틴이 호출되는 것은 아닙니다.

그렇기 때문에 `IEnumerator`변수를 할당하고 그것을 사용하게 됩니다.

```cs
using System.Collections;
using System.Collections.Generic;
using Unity.VisualScripting;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField] private float speed;
    private IEnumerator myCo;

    private void Start()
    {
        //IEnumerator 를 넣는것은 다른 함수를 참조하게 함
        //그래서 같은 함수를 넣고싶으면 변수로 지정하고 넣어야 함
        myCo = MyCo();
        StopCoroutine(myCo);
        StartCoroutine(myCo);
    }

    //코루틴은 ref out in 같은거는 안됨
    private IEnumerator MyCo()
    {
        //true가 될 때 까지 대기
        yield return new WaitUntil(() => speed > 5f);
        //false가 될 때 까지 대기
        yield return new WaitWhile(() => speed > 5f);
        print("Ok");
    }
}
```

이렇게하면, 동일한 `IEnumerator`로 코루틴을 호출할 수 있습니다.

### Action을 통해 콜백 구현

유니티의 `UnityWebRequest()` 를 사용할 때, 코루틴을 사용하여 구현하는 것을 권장합니다.

하지만, 비동기 작업을 수행하는 만큼, 해당 작업이 끝났을 때, 콜백을 해주는 것이 필요합니다. 이때, `delegate` 를 사용하여 구현할 수 있지만, 한번 쓰고말 작업에는 익명함수`Action` 을 사용할 수 있습니다.

```cs
using System;
using System.Collections;
using System.Collections.Generic;
using Unity.VisualScripting;
using UnityEngine;

public class Test : MonoBehaviour
{
    [SerializeField] private float speed;

    private void Start()
    {
        StartCoroutine(MyCo((num,model) =>
        {
            print(num);
        }));
    }

    //코루틴은 ref out in 같은거는 안됨
    private IEnumerator MyCo(Action<int,PlayerModel> _numAction)
    {
        yield return new WaitForSeconds(1f);
        _numAction(UnityEngine.Random.Range(1, 10),new PlayerModel());
    }
    public class PlayerModel
    {

    }
}
```

이런식으로 비동기 작업이 끝났을 때, 인자들을 전달하여 구현할 수 있습니다.
