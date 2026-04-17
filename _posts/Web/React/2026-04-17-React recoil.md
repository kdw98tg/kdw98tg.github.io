---
layout: post

excerpt_image: /assets/images/posts/2026-04-17-React%20recoil/file-20260417152421936.png

title: "[React/Js] Recoil을 활용해서 전역 상태 관리하기"

categories:
  -  Web
  
tags:
  - React
  - recoil

toc: true
toc_sticky: true

published: true

date: 2026-04-17
last_modified_at: 2026-04-17
---

## Recoil 이란?

![](/assets/images/posts/2026-04-17-React%20recoil/file-20260417152421936.png)

리액트 앱을 만들다 보면, 컴포넌트간 props 를 사용하여 데이터 통신을 하는 로직을 많이 사용하게 됩니다. 이게 1~2 depth 정도면 props로 데이터 통신을 할만 한데, 계층이 여러개가 되면 props로 통신을 하기가 너무 힘들어집니다. 이것을 props drilling 이라고 합니다.

이 props driiling 문제를 해결하기위해 고안된 방식이 **전역 상태 관리** 입니다.

전역에서 사용하는 데이터를 각 컴포넌트에서 사용하여 props 를 사용하지 않고,데이터를 받는 방법입니다.

그렇다면, 모든 데이터를 전역으로 관리하면 좋지 않을까? 라고 생각할 수 있습니다. 하지만 여기에는 함정이 있습니다. 전역으로 관리되는 상태 라는 것 자체가 어떤 컴포넌트든 접근을 할 수 있다는 말이 됩니다. 어떤 컴포넌트든 접근하여 데이터를 사용거나 수정하게 되면, 데이터가 쉽게 오염될 수 있고, 어디서 데이터가 오염되었는지를 파악하는데 많은 시간이 걸립니다.

또한, recoil 에서 관리되는 데이터들은 `useState`와 비슷하게, 상태가 바뀌면 화면이 리렌더링 되어야 합니다. 모든 컴포넌트에서 하나의 전역 상태를 바라보고 있다면, 데이터가 바뀌었을 때, 불필요한 컴포넌트까지 리렌더링의 대상이 될 수 있습니다.

따라서, 전역 상태를 관리할 때에는 유저의 로그인 상태 (인증 토큰, 유저정보 등), UI 테마, 다국어 설정 등의 리액트 앱을 만들다 보면, 컴포넌트간 props 를 사용하여 데이터 통신을 하는 로직을 많이 사용하게 됩니다. 이게 1~2 depth 정도면 props로 데이터 통신을 할만 한데, 계층이 여러개가 되면 props로 통신을 하기가 너무 힘들어집니다. 이것을 props drilling 이라고 합니다.

이 props driiling 문제를 해결하기위해 고안된 방식이 **전역 상태 관리** 입니다.

전역에서 사용하는 데이터를 각 컴포넌트에서 사용하여 props 를 사용하지 않고,데이터를 받는 방법입니다.

그렇다면, 모든 데이터를 전역으로 관리하면 좋지 않을까? 라고 생각할 수 있습니다. 하지만 여기에는 함정이 있습니다. 전역으로 관리되는 상태 라는 것 자체가 어떤 컴포넌트든 접근을 할 수 있다는 말이 됩니다. 어떤 컴포넌트든 접근하여 데이터를 사용거나 수정하게 되면, 데이터가 쉽게 오염될 수 있고, 어디서 데이터가 오염되었는지를 파악하는데 많은 시간이 걸립니다.

따라서, 전역 상태를 관리하면 좋은 상황은 다음과 같습니다.

1. **애플리케이션 전반에서 공통으로 의존하는 데이터**

유저의 로그인 상태(토큰, 유저 정보), UI 테마(dark / light), 다국어 설정 등의 데이터

2. **여러 페이지간에 유지되어야 하는 데이터**

여러 페이지에 걸쳐 진행되는 설문조사나 회원가입 폼, 또는 임시 데이터 (만약 SWR을 사용하고 있다면, SWR 도 캐싱을 하고 있기 때문에, 서버에 저장되는 데이터는 SWR로 관리하고, 서버에 저장되지 않지만 애플리케이션에서 전역으로 사용되어야 하는 데이터는 recoil로 관리하는것이 효율적임)

## atom 과 selector

recoil 에는 `atom`과 `selector`가 존재하게 됩니다.

우선, `atom`은 원본 데이터를 말합니다. 그 원본 데이터가 예를들어, 유저의 상태라고 생각을 해봅시다. 여기서, 유저의 정보를 출력하는 로직을 만든다고 생각해 봅시다.

그러면 모든 컴포넌트에서 원본 데이터인 `atom`을 가져와야 하고, 그 다음에 `atom`의 정보대로 특정 문구를 출력하는 함수를 만들어야 합니다.

이렇게 되면, 특정 문구가 변경될 때 마다 관리포인트는 특정 문구를 만든 컴포넌트의 개수가 될것입니다. 즉, **캡슐화**가 깨져, 유지보수에 불이익이 생길 수 있습니다.

그래서, 원본 데이터인 `atom`의 자료를 가공해주는 캡슐화된 함수인 `selector`를 지원합니다.

즉, `selector`는 원본 데이터`atom`의 데이터를 기반으로 특정 기능을 수행하게 하는 캡슐화된 함수 입니다.

거기에 더해서, 데이터를 캐싱하여, `selector`를 호출할 때, 리렌더링이 일어나지 않고 `atom`의 데이터가 변경될 때만 다시 계산을 합니다. 또한, 다른 `selector`를 참조하여 계산할 수도 있습니다.

## 사용법

### 설치

recoil 은 현재 meta 에서 유지보수가 사실상 되고 있지 않습니다. 따라서, react 19버전에는 호환되지 않습니다. recoil을 사용하신다면 react 18버전 이하로 돌려주셔야 합니다. 

```bash
npm install recoil
```

### atom 정의

store.js 라는 파일을 만들어서, atom을 정의합니다. atom 에는 해당 atom을 부를 때 사용할 key와 default 값을 넣어주면 됩니다.

```js
import { atom, selector } from "recoil";

export const userState = atom({
  key: "userState",
  default: { name: "게스트", level: 1 },
});
```

### selector 생성

위에서 만든 atom을 바탕으로 selector를 정의합니다.

```js
// 2. Selector: Atom을 기반으로 계산된 파생 데이터 (useMemo 전역 버전)
export const userInfoSelector = selector({
  key: "userInfoSelector",
  get: ({ get }) => {
    const user = get(userState); // userState를 구독
    return `${user.name}님은 현재 레벨 ${user.level}입니다.`;
  },
});

export const userLevelSelector = selector({
  key: "userLevelSelector",
  get: ({ get }) => {
    const user = get(userState);
    return user.level;
  }
});
```

### 컴포넌트에서 사용

#### 상황 1 : user의 이름을 바꾸는 상황

전역 상태에 보관된 atom 에 접근해서 유저의 이름을 바꾸는 과정입니다.

`useState`와 비슷한 역할을 하는 `useRecoilValue`를 사용해서 구현합니다.

`useRecoilValue`역시, 전역 상태가 바뀌면, 화면을 리렌더링 하게 됩니다.

```jsx
import {
  useRecoilState,
  useRecoilValue,
  useSetRecoilState,
  RecoilRoot,
} from "recoil";
import { userState, userInfoSelector } from "./store";

function Recoil1() {
  const [user, setUser] = useRecoilState(userState);
  console.log("[Recoil1] 렌더링 됨");
  console.log(user);

  return (
    <div style={{ border: "1px solid red", padding: "10px", margin: "10px" }}>
      <h3>프로필 (읽기+쓰기)</h3>
      <p>이름: {user.name}</p>
      <button onClick={() => setUser({ ...user, name: "김동우" })}>
        이름 변경
      </button>
    </div>
  );
}

export default Recoil1;


```

![](/assets/images/posts/2026-04-17-React%20recoil/file-20260417154827877.png)

![](/assets/images/posts/2026-04-17-React%20recoil/file-20260417154847079.png)

위 사진처럼, 버튼을 누르면, 값이 변경이 되고 화면이 리렌더링 되는것을 확인할 수 있습니다.

#### 상황 2

두번째는, selector 에서 계산된 값을 가져오는 로직입니다. selector에서 atom을 기준으로 만들어진 문자열을 사용하는 예시 입니다.

selector의 값을 가져올 때는 `useRecoilValue`를 사용합니다.

```jsx
import {
  useRecoilState,
  useRecoilValue,
  useSetRecoilState,
  RecoilRoot,
} from "recoil";
import { userState, userInfoSelector } from "./store"; // 위에서 만든 파일

function Recoil2() {
  const userInfo = useRecoilValue(userInfoSelector);
  console.log("[Recoil2] 렌더링 됨");
  console.log(userInfo);

return (
    <div style={{ border: "1px solid blue", padding: "10px", margin: "10px" }}>
      <h3>대시보드 (읽기 전용)</h3>
      <p>요약: {userInfo}</p>
    </div>
  );
}
export default Recoil2;

```

![](/assets/images/posts/2026-04-17-React%20recoil/file-20260417155133692.png)

#### 상황 3

세번째는, atom의 값이 변경되어도 화면에 리렌더링을 안하고 싶을때, 즉 데이터만 바꾸고 싶을때 사용하는 예제 입니다.

그럴때는 `useSetRecoilState`를 사용합니다.

useRecoilValue를 억지로 넣어서 확인을 해보는 코드를 넣었는데, 저걸 빼면, atom 의 값이 바뀌어도 화면이 리렌더링 되지 않습니다.

```jsx
import {
  useRecoilState,
  useRecoilValue,
  useSetRecoilState,
  RecoilRoot,
} from "recoil";
import { userState, userInfoSelector, userLevelSelector } from "./store"; // 위에서 만든 파일
import { useEffect } from "react";

function Recoil3() {
  const setUser = useSetRecoilState(userState);

  //atom의 데이터가 바뀌는지 확인하기 위해서 넣음
  const userLevel = useRecoilValue(userLevelSelector);
  useEffect(() => {
    console.log("중간 점검 : user의 레벨은? : ", userLevel);
  }, [userLevel]);

  return (
    <div style={{ border: "1px solid green", padding: "10px", margin: "10px" }}>
      <h3>🟢 컨트롤러 (쓰기 전용)</h3>
      {/* user 값을 직접 읽지 않고 함수 형태로 이전 상태를 가져와서 업데이트 */}
      <button
        onClick={() => setUser((prev) => ({ ...prev, level: prev.level + 1 }))}
      >
        레벨업! (렌더링 안 됨)
      </button>
    </div>
  );
}
export default Recoil3;

```

![](/assets/images/posts/2026-04-17-React%20recoil/file-20260417155509812.png)