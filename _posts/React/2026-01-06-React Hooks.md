---
title: "[react/js] React Hooks 살펴보기"

categories:
  -  React
  
tags:
  - [react, javascript, hooks]

toc: true
toc_sticky: true

published: true

date: 2026-01-06
last_modified_at: 2026-01-06
---

## 리액트 Hooks

리액트에서는 컴포넌트의 상태 및 생명주기를 관리하기 위해 나온 개념입니다. useState를 사용하면, 컴포넌트가 계속 리랜더링 되는데, 여기서 이벤트 같은 녀석들은 컴포넌트가 리랜더링 되면, 메모리에 계속 쌓이는 구조이기 때문에, 계속 호출할 필요가 없고, 컴포넌트를 다 쓰고 나서 해제될 때는 메모리에서 해제해주는 과정이 필요합니다. 이런 일련의 과정들을 수행하기 위해 나온 개념이라고 보면 됩니다.

Hooks는 10가지정도 있는데, 보통 잘 사용하는 것이 3가지 정도(useState, useEffect, useCallback)입니다.

예제를 통해 살펴보겠습니다. 우선, 전체 코드 입니다.

```js
import { useCallback, useEffect, useState } from 'react';

function Counter() {
    console.log("Render Counter!");

    const [value, setValue] = useState(0);
    useEffect(() => {
        console.log("컴포넌트가 최초 마운트 될 때 호출 1");

        const clickBodyEvent = () => {
            console.log('click body');
        }
        document.body.addEventListener('click', clickBodyEvent);

        //클린업 함수
        return () => {
            console.log("컴포넌트가 언마운트 될 때 호출 1");
            document.body.removeEventListener('click', clickBodyEvent);
        };
    }, []);

    useEffect(() => {

        console.log("컴포넌트가 최초 마운트 될 때 호출 2");

        return () => {
            console.log("컴포넌트가 언마운트 될 때 +dependency list에 있는 값이 바뀔때, 호출 2");
        };

    }, [value]);

    const increaseValue = useCallback(() => {
        setValue(value + 1);
    }, [value]);

    const resetValue = useCallback(() => {
        setValue(0);
    }, [])

    return (
        <div>
            <h1>value: {value}</h1>
            <button onClick={increaseValue}>Increase value</button>
            <button onClick={resetValue}>Reset value</button>
        </div>
    );
}

export default Counter;
```


### useEffect

우선, useEffect부터 살펴 보겠습니다. useEffect는 컴포넌트가 리랜더링 될 때, 불필요한 작업을 하지 않게 하기 위해서 만들어진 Hooks 입니다. 예를들어, 컴포넌트가 리랜더링 될 때마다 이벤트가 재설정되는 경우가 이에 해당됩니다.

컴포넌트가 마운트 될 때, 딱 한번 실행되도 괜찮은 기능을 구현할 때 사용합니다.


```js
 useEffect(() => {
        console.log("컴포넌트가 최초 마운트 될 때 호출 1");

        const clickBodyEvent = () => {
            console.log('click body');
        }
        document.body.addEventListener('click', clickBodyEvent);

        //클린업 함수
        return () => {
            console.log("컴포넌트가 언마운트 될 때 호출 1");
            document.body.removeEventListener('click', clickBodyEvent);
        };
    }, []);
```

useEffect는 2가지로 구성되어 있습니다. 컴포넌트가 마운트 될 때, 동작을 하는 구간, 컴포넌트가 언마운트 될 때 동작을 하는 구간 이렇게 나뉩니다;.

return 키워드 다음에 정의된 함수가 언마운트 할 때 호출되는 함수이며, 보통 이벤트를 해제할 때 사용합니다.

다음으로 주목해야 하는 부분은, useEffect의 두번째 인자로 들어오는 배열 입니다.

```js
    useEffect(() => {

        console.log("컴포넌트가 최초 마운트 될 때 호출 2");

        //클린업 함수임
        //컴포넌트가 언마운트 될 때 호출됨
        return () => {
            console.log("컴포넌트가 언마운트 될 때 +dependency list에 있는 값이 바뀔때, 호출 2");
        };

    }, [value]);
```

만약 두번째 인자에, useState로 정의된 값이 들어온다면, 동작이 조금 다릅니다. 배열의 값이 바뀔 때 마다 useEffect가 호출되게 됩니다. 그리고, 리랜더링이 일어날 때, 클린업 함수가 먼저 호출되고, 그 다음에 useEffect에 정의된 함수가 호출이 됩니다.

즉, 순서는 다음과 같습니다.

1. 컴포넌트가 마운트 될 때, useEffect 함수 호출
2. 그 다음부터 리랜더링 될 때, 클린업 함수 호출, 그다음에 useEffect 함수 호출
3. 컴포넌트가 언마운트 될 때, 클린업 함수 호출

만약, 리스트에 값이 들어가게 되면, useEffect는 배열 안에 있는 값에 의존성을 가지며 동작하게 됩니다. 그래서 이 배열을 `dependency list`라고 부릅니다.

### useCallback

useCallback은 특별한 동작 타이밍이 있는것은 아닙니다. 이 기능은 리랜더링시 성능 최적화를 위해 사용되는 개념 입니다.

```js
    const increaseValue = useCallback(() => {
        setValue(value + 1);
    }, [value]);

    //근데 리셋같은경우는 0으로 초기화하는게 무조건 똑같으니까 useCallback을 사용하면 좋음
    const resetValue = useCallback(() => {
        setValue(0);
    }, [])

    return (
        <div>
            <h1>value: {value}</h1>
            <button onClick={increaseValue}>Increase value</button>
            <button onClick={resetValue}>Reset value</button>
        </div>
    );
```

useCallback은 리랜더링 시, 불필요하게 함수가 생성되는것을 막고자 나왔습니다. 그리고, 그 뒤에 오는 `dependency list`에는 어떤 값이 바뀔 때 함수가 생성되어야 하는지를 정의합니다.

지금 value를 바꾸는 함수는 value가 바뀜에 따라 컴포넌트가 리랜더링 되며, 계속 함수를 만들어줘야 다음 value값에 +1 을 할 수 있을 것입니다.

사실 그래서 value를 증가하게 하는 로직에서는 사용할 필요가 없습니다.

하지만 value의 값을 무조건 0으로 리셋시켜야 하는 함수에서는 이야기가 다릅니다. value의 값을 0으로만 세팅하는것은 변하지 않기 때문입니다. 따라서 해당 로직에 useCallback을 사용해서 불필요하게 함수가 계속 생성되는것을 막을 수 있습니다.

