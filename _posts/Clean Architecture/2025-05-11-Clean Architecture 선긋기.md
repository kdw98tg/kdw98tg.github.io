---
title: "[Clean Architecture] 선긋기"

categories:
  -  Clean Architecture
  
tags:
  - [Clean Architecture]

toc: true
toc_sticky: true

published: true

date: 2025-05-11
last_modified_at: 2025-05-11
---

## 소프트웨어 아키텍쳐란

저자가 말하는 소프트웨어 아키텍쳐란 다음과 같습니다.

> 소프트웨어 아키텍처는 선을 긋는 기술이며, 나는 이러한 선을 경계라고 부른다.

경계는 소프트웨어 요소들을 서로 분리하고, 경계 한편에 있는 요소가 반대편에 있는 요소를 알지 못하도록 막는 역할을 합니다. 이것은 앞서 이야기한 `결정을 최대한 늦추기`위해서 입니다. 서비스 또는 시스템의 초기에는 서비스를 하면서 어떤 상황이 벌어질지, 사용자가 얼마나 될지, 서비스가 어떻게 발전해나갈지를 알 수 없기 때문에, 세부사항에 대한 결정을 최대한 늦추는 방향으로 설계가 되어야 합니다. 그래야 지금은 Mysql 데이터베이스를 쓰다가도 Mongo 데이터베이스를 사용하도록 바꾸는데 많은 자원을 쏟지 않아도 되기 때문입니다.

결국, 소프트웨어에서 많은 자원을 쓰게 되는것은, 바로 강한 결합 입니다. 강한 결합으로 세부사항들을 묶어두게 되면, 어떤 모듈이 바뀔때 마다 에러가 수십개 ~ 수백개가 뜰것이고, 이것을 다 고치는데에는 엄청난 노력과 시간이 들어가야 합니다. 따라서, 선을 잘 그어서 소프트웨어의 세부사항을 결정하는데 유연함을 추가해야 합니다.

## 선긋기

그렇다면, 어떻게 선을 그어야 하는것일까요? 당연한 이야기겠지만, 관련이 있는 것과 없는 것 사이에 선을 긋습니다. GUI, 업무규칙, 데이터베이스 등등 사이에 경계를 만들 수 있습니다.

![](/images/Pasted%20image%2020250511183345.png)

위 그림은, 업무 규칙과, 데이터베이스를 나눈 모습입니다. 업무규칙과 데이터베이스가 아무리 연관이 없다고 해도, 결국 하나의 소프트웨어에서 동작할 때는 이어져 있어야 합니다. 여기서 느슨한 결합을 만들기 위해 인터페이스로 `의존성 역전의 원칙`을 구현합니다. 만약 데이터베이스가 바뀌어야 한다면, Database Access 모듈만 갈아끼우면 됩니다.

이렇게 한다면, 어떤 이점이 있을까요? 일단 제 경험을 토대로 이야기를 해본다면, 업무 규칙과 데이터베이스를 동시에 구현하게 될때, 데이터베이스가 말썽을 부리면 업무규칙을 구현하는데 상당한 시간이 걸리게 됩니다. 하지만, 지금처럼 선을 잘 그어서 인터페이스로 소통하게 된다면, 그냥 휘발성 메모리 (예를들어 List<>)를 사용하여 업무규칙을 만드는데 집중하고, 업무규칙이 완성된다면 그 뒤에 데이터베이스를 결정하여 작업을 할 수 있게 됩니다. 이것이 테스트나 데이터베이스를 갈아끼우거나 등등의 작업에서 효율이 좋습니다.

## 플러그인 아키텍쳐

지금까지, 연관이 없는 기능들을 분리해서 선을 긋고, 그 선 사이에 인터페이스를 넣어서 세부사항(저수준 모듈)들을 최대한 나중에 구현하게 하는것을 하였습니다. 세부사항을 언제든지 갈아끼울 수 있다는 측면에서 해당 아키텍쳐는 플러그인 아키텍쳐라고 부릅니다.

![](/images/Pasted%20image%2020250511183948.png)

소프트웨어 개발 기술의 역사는 플러그인을 쉽게 생성, 확장, 유지보수하도록 아키텍쳐를 확립할 수 있게 만드는 방법에 대한 이야기라고 합니다. 이러한 확장성 때문에, IDE에 자신만의 플러그인을 구성하여 자신만의 환경을 만들거나, 스타크래프트 또는 마인크래프트같은 게임에 유즈맵을 만들어 플레이 하거나, 프레임워크(Spring, Unity, Android 등)에 본인이 만들고싶은 서비스의 세부사항을 만들어 자신만의 서비스를 만들 수 있게 되었습니다.