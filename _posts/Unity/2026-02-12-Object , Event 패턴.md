---
title: "[Unity] Unity의 GameObject와 행동의 분리"

categories:
  -  Unity
  
tags:
  - [Unity, Design Pattern]

toc: true
toc_sticky: true

published: true

date: 2026-02-12
last_modified_at: 2025-02-12
---

이번에 공포게임을 만들고, 리팩터링을 고민하다가 선배 개발자로부터 피드백을 받은 내용을 바탕으로 간단한 예제를 만들었습니다.

피드백의 내용은 다음과 같습니다.

> 지금 만든 공포게임은 어떤 오브젝트가, 어떤 행동을 한다로 거의 다 정의가 된다.
> 그렇다면, 특정 오브젝트에 특정 행동을 정의하면 같은 로직이 반복되고, 유지보수가 어렵다.
> 오브젝트는 껍데기만 놔두고, 특정 행동을 캡슐화 하여 오브젝트에 끼워넣으면 관리가 좋다.

이것을 듣고 무한한 확장성에 충격을 받고 좀 검색을 해봤습니다. 

게임 오브젝트를 빈 껍데기로 쓰고, 로직을 갈아끼우는 방식은 이미 유니티에서 만들어둔 ECS (Entity Component System)과 닯아 있었습니다.

ECS 란, Entity, Component, System 3가지 구성요소로 프로그래밍을 하는 기법 입니다. Entity는 그저 빈 껍데기, Component는 데이터만 담고 있는 주머니, System은 실제 로직을 처리하는 엔진 이라는 뜻을 내포하고 있습니다.

여튼, 이런 식으로 게임을 구성하면, 공통되는 로직들을 그냥 오브젝트에 끼워넣기만 하면 됩니다. 즉, 다음과 같은게 가능해 집니다.

1. 축구공이 앞으로 이동하다가 펑! 터진다.
2. 농구공이 앞으로 이동하다가 펑! 터진다.
3. 리모컨이 앞으로 이동하다가 펑! 터진다.
4. 컴퓨터가 앞으로 이동하다가 펑! 터진다.
5. 세면대가 앞으로 이동하다가 펑! 터진다.

위 로직들은 지금의 공포게임 구조로는 축구공, 농구공, 리모컨, 컴퓨터, 세면대 오브젝트에 일일이 기능들을 구현해야 했습니다. 

하지만 선배 개발자가 피드백해준 방식으로 풀면, 축구공, 농구공, 리모컨, 컴퓨터, 세면대 등의 오브젝트는 무엇이 되든 상관이 없고, `이동하다가 펑! 터진다` 라는 로직한 하나 정의 한 다음 빈 껍데기인 오브젝트에 넣어주면 됩니다.

이렇게 되니, 어떤 오브젝트던, 조건만 갖추면 특정 기능을 할 수 있게 되었습니다.

예제가 조금 길어서, 예제는 깃허브 링크를 남겨두겠습니다. 아래의 깃허브 주소에 `ObjectManager1` 폴더를 찾아 보시면 됩니다.

https://github.com/kdw98tg/UnityStudy

## 예제

간단하게 Event (행동) 과 Object (빈껍데기)를 구현한 예제를 보여드리겠습니다. Player가 이동하는 로직을 위 패턴으로 구현한 예제입니다.

Event_MoveWithTransform.cs
```cs
using System.Collections;
using UnityEngine;

namespace ObjectManager1
{
    public class Event_MoveWithTransform : EventBase
    {
        public override void Init()
        {
            base.Init();
            id = EventId.Event_MoveWithTransform;
        }

        public override void Execute(ObjectBase _objectBase, EventDataBase _data)
        {
            EventData_MoveWithTransform data = _data as EventData_MoveWithTransform;
            _objectBase.transform.position += data.Direction * data.Speed * Time.deltaTime;
        }
    }
}
```

위 코드는 추상화된 Object 를 받아서, 그 오브젝트의 위치를 참조하여 앞으로 움직이게 하는 코드 입니다. 여기에 ObjectBase를 상속받는 그 어떤객체가 들어와도 잘 동작하게 됩니다.

Object_Player.cs
```cs
using ObjectManager1;

public class Object_Player : ObjectBase
{
    public override void Init()
    {
        base.Init();
        id = (int)ObjectId.Player;
    }
}
```

위 코드는 단지 ID만 붙어 있는 오브젝트 입니다. 즉, 빈 껍데기 입니다.

Player.cs
```cs
using UnityEngine;

namespace ObjectManager1
{
    public class Player : MonoBehaviour
    {
        private EventData_MoveWithTransform moveEventData = null;

        [SerializeField] private float speed = 1f;


        private void Awake()
        {
            moveEventData = new EventData_MoveWithTransform();
        }

        public void Init()
        {

        }

        public void Move(ObjectId objectId, Vector3 moveDirection)
        {
            moveEventData.Direction = moveDirection;
            moveEventData.Speed = speed;

            CommandInvoker<DoActionParam>.Execute(new DoActionParam
            {
                ObjectId = objectId,
                EventId = EventId.Event_MoveWithTransform,
                EventData = moveEventData,
            });
        }
    }
}
```

위 코드는 Player의 행동을 정의하는 일종의 컴포지션 클래스 입니다. 위 코드에서, Move에 필요한 데이터를 넣은 다음, GameManager에게 어떤 오브젝트를 어떻게 움직여라, 라는 커맨드를 보내주면, GameManager에서 알아서 그 오브젝트를 찾아서 처리하게 됩니다.

GameManager.cs
```cs
using System;
using Unity.VisualScripting;
using UnityEngine;
namespace ObjectManager1
{
    public class GameManager : MonoBehaviour
    {
        private ObjectManager objectManager = null;
        private EventManager eventManager = null;

        private void Awake()
        {
            objectManager = GetComponentInChildren<ObjectManager>();
            eventManager = GetComponentInChildren<EventManager>();
        }

        private void Start()
        {
            InitCommands();

            eventManager.Init();
            objectManager.Init();
        }

        private void DoAction(DoActionParam _param)
        {
            var eventBase = eventManager.GetEvent(_param.EventId);
            objectManager.DoAction(_param.ObjectId, eventBase, _param.EventData);
        }

        private void InitCommands()
        {
            CommandInvoker<DoActionParam>.Add(new Command_DoAction(DoAction));
        }
    }
}
```

이렇게 하면, 어떤 오브젝트든, 행동을 주입받으면 그대로 움직이게 할 수 있습니다.


## 마무리

우선, 지금은 Event (행동)객체가 MonoBehaviour를 상속받아서, 게임오브젝트로 나와있습니다. 여기서 MonoBehaviour의 행동을 받지 않는데, 굳이 리소스를 낭비할 필요가 없다고 생각됩니다. 우선 놔둔것은 코루틴을 돌리는 로직이 필요할까봐서 인데, 일단 이렇게 사용하다가 더이상 필요가 없어진다고 하면 일반 C# 클래스로 빼는게 좋아보입니다.

이 방법을 들었을때, 무한한 확장성에 새로운 시야가 트인 느낌이었습니다. 만드는데는 한 2일정도 걸린거 같습니다. 처음에는 어떻게 구현할까 했는데, 막상 구현해보니 또 간단해 보이네요. 

또, 느낀것은 지식의 전달 입니다. 문어와 사람은 기본 지능은 비슷하다고 합니다. 하지만 사람은 문명을 이루었고, 문어는 아직 사람에게 잡아먹히고 있습니다. 이 차이는 바로, `지식의 전달`에서 온다고 생각합니다.

사람은 선배의 지식을 후배에게 전달할 수단 (말, 글 등)이 있고, 문어는 지식을 전달할 수단을 가지고 있지 않습니다. 따라서 문어는 오랜 시간에 걸친 경험과 지식이 자기 세대에서 끝이 나는것이죠.

이 패턴을 들었을 때, 그런 느낌이 들었습니다. 제가 혼자 고민했더라면, 꽤 오랜시간에 걸쳐 이 해답을 찾았을거 같습니다. 그 시간을 아끼며 한층 더 발전한것 같습니다. 선배 개발자에게 감사를 표합니다.