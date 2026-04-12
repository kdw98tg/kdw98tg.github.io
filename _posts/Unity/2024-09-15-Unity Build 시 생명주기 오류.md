---
title: "[Unity] Unity 빌드시 Editor 에서는 작동하는데, Build 하면 Null 이 되는 오류"

categories:
  -  Unity
  
tags:
  - [Unity, Error, Build, LifeCycle]

toc: true
toc_sticky: true

published: true

date: 2024-09-15
last_modified_at: 2024-09-15
---

## 생명주기 오류

### 생명주기 순서 보장 안됨 오류

에디터에서 작업을 하고, 이제 빌드를 해서 테스트를 해보려고 했는데, 예상치 못한 문제가 발생하였습니다. 

`Android` 타겟으로 빌드를 진행하였는데, 에디터에서는 분명 로직이 돌아가는데, 빌드만 하면 작동이 안되는 문제가 발생하였습니다.

지금 아래의 그림을 보시면 Prefab으로 관리되고 있는 객체들이 있는데, 처음에는 이것들이 Addressable 이라 Load 되기 전에 호출해서 안되는 줄 알았습니다.

하지만, Editor 에서는 잘 동작하고, Addressable 로 Load 하는게 아니라 씬에 그냥 박아놨기 떄문에 이 문제는 아니었습니다.

![](/images/Pasted%20image%2020240915115257.png)

그러다가 생명주기의 문제인가? 싶어가지고 코드를 살짝 변경해 보았습니다. 

![](/images/Pasted%20image%2020240915115906.png)

위의 코드를 아래처럼 바꾸었더니 동작하였습니다.

![](/images/Pasted%20image%2020240915120003.png)


### 해결

문제는 바로 `Awake` 함수의 호출 순서를 유니티가 보장하지 못하여서 그렇습니다. `yesNoDialog`에서는 `Awake` 에서 필요한 변수들을 초기화 시켜주도록 만들어 놨습니다.

![](/images/Pasted%20image%2020240915120123.png)

`IntroScene` 스크립트에서 yesNoDialog 를 초기화 하고, 바로 SetAcrtive(false)로 비활성화 시켜주는데, `yesNoDialog`의 `Awake` 가 미쳐 돌지 않은 상태에서 비활성화를 시켜주니, 변수들이 할당되지 않아 `NullPointerException` 이 나타났습니다.

이는 Android Os 에서 Unity 프로젝트를 관리하게 되면서, 다른 로직이 개입되어 발생하는 문제라고 합니다. 그렇기 때문에, Awake 에서는 변수 할당만을 처리해 주고, 해당 변수를 사용하는 로직은 그 이후에 호출되는 함수에서 처리하는것이 좋습니다. 또는 `Lazy Init` 기법을 활용해 초기화 순서를 무조건 보장해주는 것이 해답이라고 할 수 있습니다.