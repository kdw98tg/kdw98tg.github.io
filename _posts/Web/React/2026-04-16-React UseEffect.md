---
layout: post

title: "[React/Js] React Useffect 살펴보기"

categories:
  -  Web
  
tags:
  - React
  - JavaScript
  - Hooks
  - UseEffect

toc: true
toc_sticky: true

published: true

date: 2026-04-16
last_modified_at: 2026-04-16
---

## 시작하며

React FrameWork 만의 특징이라고 하면 `Hook`을 말 할 수 있습니다. 기존의 React는 Class형 컴포넌트로 구성되어 있었지만, 현재는 함수형 컴포넌트로 구성이 되어 있습니다.

Class 는 상속 및 오버라이드 기능을 통해서 생명주기를 정할 수 있지만, 함수는 그것이 안됩니다. Class는 Instantiate 를 하면, Heap 메모리에 상주하며 상태를 저장할 수 있지만, 함수는 호출될 때 마다 함수 안에 있는 지역변수가 초기화 되어버려서 상태를 유지할 수 없다는 문제가 존재 하였습니다.

이러한 문제를 해결하기 위해서 등장한 개념이 바로 `Hook` 입니다. 함수형 컴포넌트에서도 상태를 저장하거나 관리할 수 있도록 만들어주는 기술 입니다.

즉, 클래스에 변수를 사용하듯이 함수형 컴포넌트에서 특정 값을 저장하거나 변경하려면 이 `Hook`사용이 필수 입니다. 

저는 지금까지 컴파일언어를 많이 사용했기 때문에, 함수형으로 관리되는 컴포넌트가 생소하였습니다. 그래서 이 `Hook`들을 정리하며 언제 어디서 쓸 수 있는지에 대한 내용을 작성합니다.

## UseEffect

React라는 Framework 가 나온 배경에는 여러가지 이유가 있겠지만, 그중, 정적인 html의 랜더링 과정을 동적으로 구현하기 위함도 있습니다. 이 과정을 다시 그린다 즉, 리렌더링 이라고 부릅니다.

함수형 컴포넌트라는것은 결국 함수이기 때문에, 상태를 바꾸려면, 이 함수를 다시 호출하여 값을 바꿔준 후, 화면에 렌더링 하는 과정을 거치게 됩니다.

이때, 함수에 정의된 모든 내용들이 실행됨으로, 비효율이 발생하게 됩니다. 

예를들어, 숫자를 하나 올릴 때, 숫자에 관련된 로직들만 동작하게 하고 싶은데, 이름과 관련된 로직들도 같이 동작을 해버리는것이죠.

이러한 문제를 해결하기 위 해 나온것이 `UseEffect` hook 입니다.

`UseEffect`는 컴포넌트가 랜더링될 때마다, 특정한 작업을 수행하도록 설정하는 Hook 입니다.

쉽게 말해서, 컴포넌트가 화면에 나타났을 때(Mount), 화면에서 사라졌을 때(Unmount) 혹은 특정 데이터가 변했을 때(Update) 특정 행동을 해줘! 라고 명령할 수 있습니다.

## 예제

### 기본 사용

나이와 이름을 입력하는 `input`이 있다고 가정합니다. 처음 폼에 진입했을 때, "이름과 숫자를 입력해 주세요!" 라는 메세지가 뜨고, 그 이후로 나이나 이름이 바뀔 때 마다 특정 로그를 찍어준다고 가정합니다.

우선, `UseState`훅만 써서 구현한 예시 입니다.

```jsx
import { useState } from "react";
function UseEffect1() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");

  console.log("이름과 숫자를 입력해 주세요.");
  console.log("이름이 변경되었습니다.");
  console.log("숫자가 변경되었습니다.");

  return (
    <div>
      <h1>이름</h1>
      <input type="text" value={name} onChange={(e) => setName(e.target.value)} />
      <h1>숫자</h1>
      <input type="number" value={count} onChange={(e) => setCount(e.target.value)} />
    </div>
  );
}

export default UseEffect1;

```

![](/assets/images/posts/Pasted%20image%2020260416101509.png)

위 이미지를 보면, 컴포넌트가 렌더링 되자 마자, console.log가 3개가 뜨는것을 확인할 수 있습니다. 이러면 비효율이 발생하고, `alert`같은 동작이 일어난다고 하면, 컴포넌트가 마운트 되자마자, 사용자는 팝업창을 3개를 보게 됩니다.

![](/assets/images/posts/Pasted%20image%2020260416093425.png)

또한, 숫자만 변경했는데, 계속 로그가 3개씩 찍히는것을 볼 수 있습니다.

이러한 동작을 막기 위해서, `UseEffect`훅을 사용할 수 있습니다.

```jsx
import { useEffect, useState } from "react";
function UseEffect1() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");

  useEffect(() => {
    console.log("이름과 숫자를 입력해 주세요.");
  }, []);

  useEffect(() => {
    console.log("이름이 변경되었습니다.");
  }, [name]);

  useEffect(() => {
    console.log("숫자가 변경되었습니다.");
  }, [count]);

  return (
    <div>
      <h1>이름</h1>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <h1>숫자</h1>
      <input
        type="number"
        value={count}
        onChange={(e) => setCount(e.target.value)}
      />
    </div>
  );
}

export default UseEffect1;

```

`UseEffect`는 2개의 인자를 받는데, 첫번째 인자로는, 어떤 행동을 할것인가? 에 대한 내용이고, 두번째 인자는 `Dependency Array`라고 해서, 배열 안에 있는 값이 변경될 때만, 첫번째 인자로 넣은 행동을 수행하게 할 수 있습니다.

이렇게 하면, 저희가 원하는 대로 이름을 바꾸면, 이름을 바꾸었다라는 로그가, 숫자를 바꾸면 숫자를 바꾸었다는 로그가 뜨게할 수 있습니다.

지금 조금 이상한 것이 제일 처음 마운팅 되었을 때, 모든 로그가 다 뜬다는 점인데, 이것은 `UseRef`라는 훅을 사용하여 동작을 수정할 수 있습니다. 해당 내용은 `UseRef`에 대한 포스팅에서 다루겠습니다.

![](/assets/images/posts/Pasted%20image%2020260416093746.png)

### Clean Up 함수 사용

이번에는 CleanUp 함수를 사용해 보겠습니다. 

만약, `setInterval`과 같은 함수가 있다고 생각해 보겠습니다. 이 함수는 동작이 시작되면, 메모리 어딘가에 생성되어 특정 시간마다 같은 반복을 수행하도록 설계되어 있습니다.

하지만, 위에서 살펴보았던 `UseEffect`는 마운트 될 때마다 수행된다고 하였습니다. 그럼 이 두개가 만나게 되면, 마운트될 때 마다 `setInterval` 함수가 실행되어 메모리를 계속 점유하게 되고, 저희가 원하지 않는 동작이 수행될 뿐만 아니라, 언젠가 메모리가 다 차게되면 프로그램이 멈춰버릴 겁니다.

이때, 특정 이벤트나 동작을 컴포넌트가 언마운트 될 때, 해제시켜주는 동작이 바로 `Clean Up` 입니다.

아래와 같은 Timer 컴포넌트가 있다고 가정합니다.
```jsx

function Timer() {
  setInterval(() => {
    console.log("Timer 돌아가는 중 ...");
  }, 1000);
  return <div>Timer Start!</div>;
}
export default Timer;

```

``` jsx
import { useState } from "react";
import Timer from "./Timer";
function UseEffect2() {
  const [showTimer, setShowTimer] = useState(false);
  return (
    <div>
      {showTimer && <Timer />}
      <button onClick={() => setShowTimer(!showTimer)}>
        {showTimer ? "Hide Timer" : "Show Timer"}
      </button>
    </div>
  );
}

export default UseEffect2;

```

![](/assets/images/posts/Pasted%20image%2020260416095320.png)

위처럼 코드를 짜고, Timer를 돌려보면, 첫번째 타이머를 돌릴때는 정상작동을 하나, 버튼을 여러번 누르면 Timer의 setInterval 이 중복이 되어 엄청 빠르게 로그가 찍히는것을 볼 수 있습니다.

이것을 막기 위해서 UseEffect의 CleanUp 기능을 사용합니다.

```jsx
import { useEffect } from "react";

function Timer() {
  useEffect(() => {
    const interval = setInterval(() => {
      console.log("Timer 돌아가는 중 ...");
    }, 1000);
    return () => clearInterval(interval);
  }, []);
  return <div>Timer Start!</div>;
}
export default Timer;

```

이런식으로 컴포넌트가 언마운트될 때, Interval을 해제해주면 됩니다. 그러면 정상작동을 하게 됩니다.