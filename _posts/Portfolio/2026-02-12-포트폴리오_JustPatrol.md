---
title: "[포트폴리오] Unity-JustPatrol"

categories:
  -  Portfolio
  
tags:
  - [Portfolio, JustPatrol]

toc: true
toc_sticky: true

date: 2026-02-12
last_modified_at: 2026-02-12
---

## 프로젝트 내용

`JustPatrol`은 3D 공포게임 입니다. 학교의 야간 경비에게 벌어지는 기이한 일들을 담았습니다. (스팀에 출시가 되어 있어, 자세한 플레이 영상은 생략합니다.)

![](/images/just%20patrol%20icon.png)

![](/images/TitleScene.png)

![](/images/SecurityRoom.png)

![](/images/Opend%20Window.png)

![](/images/First%20Floor.png)

## 기술 스택

- Unity 6.0 (6000.0.64f1) / C#
- Platform : WIndows
- Store : Steam

## 기술적 내용

- MonoBehaviour의 커플링을 최대한 없애고자, 각 도메인마다 추상화를 사용함
- SingleTon 사용을 최대한 지양하고, Command 패턴을 사용해서 이벤트 기반 구조로 관리함
- Localization Text를 사용하여 영어, 한국어에 대응함
- 확장성을 고려한 공포 이벤트 요소를 관리하는 시스템
- Object 에 Event를 끼워넣는 ECS(Entity Component System) 방식의 설계

## 배운점

- 3인 팀으로 진행하며, 팀장을 맡았음. 3명의 의견을 최대한 수렴하며 프로젝트를 진행하려고 하였음. 하지만, 잡음도 너무 많아지고 통일성이 없어지는 결과를 낳았음. 어느정도는 팀장이 이끌고 가야한다라는것을 느낌
- 프로토타이핑 없이 바로 게임 제작에 들어갔음. 핵심 재미요소를 잘 정한것 같아, 프로토타이핑을 하는것이 시간낭비라고 생각했음. 하지만, 재미요소를 제대로 검증하지 않고 프로젝트를 완성해 보니, 처음 생각했던것과는 결과가 많이 달랐음. 프로토타이핑의 중요성을 깨달았음
- 유니티는 자유도가 높음 프레임워크임. 유니티만 하면, 구조에 매몰되는 습관이 있었음. 하지만, 결국 사용자들은 코드의 구성을 모르기 때문에, 큰 문제가 되지 않는다면 먼저 프로젝트를 완성하고 완성본을 가지고 구조를 고민해보는것이 정답이라는것을 깨달음