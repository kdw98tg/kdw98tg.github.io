---
layout: post

title: "[Unity] Unity에서 CommandPattern 구현하기"

categories:
  -  Unity
  
tags:
  - [Unity, DesignPattern, CommandPattern]

toc: true
toc_sticky: true

published: true

date: 2024-09-01
last_modified_at: 2024-09-01
---

## Command 패턴 이란?

다들 철권 이라는 게임을 한번쯤은 들어보셨을 겁니다. 이 게임은 유저의 복잡한 입력들의 순서를 기억했다가, 게임속의 케릭터가 해당 입력들의 순서대로 적을 공격합니다. 또한, 리그오브레전드 라는 게임에서는 '다시보기' 기능을 지원하여 플레이어가 언제 어떤 행동을 했는지를 순서대로 보여주게 됩니다.

이러한 기능을 구현하기 위해 사용하는 것이 `Command 패턴` 입니다.

> Command 패턴 이란, 실행될 기능을 캡슐화 함으로써 주어진 여러 기능을 실행할 수 있는 재사용성이 높은 클래스를 설계하는 패턴 입니다.
> 
> 즉, 요청을 객체화 한다고 생각하면 됩니다.

해당 패턴을 구현하기 위해서는 `ICommand`라는 인터페이스를 만들고, 실행될 행동을 class 로 객체화 시켜준 다음, `ICommand` 를 상속하여 메서드를 구현시켜 줍니다.

## 플레이어가 움직이는 행동을 캡슐화 하기

이번에 만들어볼 예제는 UnityKorea 에서 제공하는 에셋에 담겨있는 내용입니다. 플레이어가 움직이는 방향과 위치를 캡슐화 하고, 이를 `Stack` 자료구조에 담아서, `Redo`와 `Undo`를 수행할 수 있도록 합니다.

### Player 의 움직임 구현

우선,  `PlayerMove`라는 스크립트를 구현하여, Player의 움직임을 구현하도록 합니다.

🗅 **<span style="color: #c03a92">class PlayerMove</span>**
```csharp
using UnityEngine;

namespace Command
{
    public class PlayerMove : MonoBehaviour
    {
        public void Move(Vector3 _movement)
        {
            Vector3 destination = transform.position + _movement;
            transform.position = destination;
        }
    }
}
```
위의 움직임은 방향벡터만 더하여, 플레이어가 1 씩 움직이도록 하였습니다.

### Player의 입력 구현

Player 의 입력을 스크립트로 구현해 줍니다., 또한 여기서 Command 를 실행시킬 함수 `RunPlayerCommnad`를 정의해 줍니다.

해당 메서드는 `ICommand`타입의 클래스를 받아서 각자 정의된 행동을 실행할 수 있도록 해줍니다.

🗅 **<span style="color: #c03a92">class PlayerInput</span>**
```csharp
using UnityEngine;

namespace Command
{
    public class PlayerInput : MonoBehaviour
    {
        [SerializeField] private KeyCode forwardKey = KeyCode.W;
        [SerializeField] private KeyCode backKey = KeyCode.S;
        [SerializeField] private KeyCode leftKey = KeyCode.A;
        [SerializeField] private KeyCode rightKey = KeyCode.D;
        [SerializeField] private KeyCode undoKey = KeyCode.U;
        [SerializeField] private KeyCode redoKey = KeyCode.R;

        private PlayerMove player = null;

        private void Awake()
        {
            player = GetComponent<PlayerMove>();
        }

        private void Update()
        {
            // Check for forward movement key
            if (Input.GetKeyDown(forwardKey))
            {
                OnForwardInput();
            }

            // Check for back movement key
            if (Input.GetKeyDown(backKey))
            {
                OnBackInput();
            }

            // Check for left movement key
            if (Input.GetKeyDown(leftKey))
            {
                OnLeftInput();
            }

            // Check for right movement key
            if (Input.GetKeyDown(rightKey))
            {
                OnRightInput();
            }

            // Check for undo key
            if (Input.GetKeyDown(undoKey))
            {
                OnUndoInput();
            }

            // Check for redo key
            if (Input.GetKeyDown(redoKey))
            {
                OnRedoInput();
            }
        }
        private void OnLeftInput()
        {
            RunPlayerCommand(player, Vector3.left);
        }

        private void OnRightInput()
        {
            RunPlayerCommand(player, Vector3.right);
        }

        private void OnForwardInput()
        {
            RunPlayerCommand(player, Vector3.forward);
        }

        private void OnBackInput()
        {
            RunPlayerCommand(player, Vector3.back);
        }

        private void OnUndoInput()
        {
            CommandInvoker.UndoCommand();
        }

        private void OnRedoInput()
        {
            CommandInvoker.RedoCommand();
        }

        private void RunPlayerCommand(PlayerMove _playerMover, Vector3 _movement)
        {
            if (_playerMover == null) return;

            ICommand command = new MoveCommand(_playerMover, _movement);

            CommandInvoker.ExecuteCommand(command);
        }
    }
}
```

해당 클래스에서 아직 구현되지 않은 `ICommand`, `CommandInvoker`, `MoveCommand`를 정의 해봅시다.

### ICommand 구현

`ICommand` 인터페이스를 구현해 봅시다. `ICommand`인터페이스는 실행을 담당하는 `Execute()`와 물리기를 담당할 `Undo()` 메서드를 가지고 있습니다.

🗅 **<span style="color: #c03a92">interface ICommand</span>**
```csharp
namespace Command
{
    public interface ICommand
    {
        public void Execute();
        public void Undo();
    }
}
```

### MoveCommand 구현

해당 클래스는 `ICommand`인터페이스를 상속받아 구현하는 구현 클래스 입니다. 이 클래스는 Player가 움직이는 행동을 캡슐화 하여 정의 합니다.

🗅 **<span style="color: #c03a92">class MoveCommand</span>**
```csharp
using UnityEngine;
namespace Command
{
    public class MoveCommand : ICommand
    {
        private PlayerMove playerMover = null;
        private Vector3 moveVector = Vector3.zero;

        public MoveCommand(PlayerMove _playerMover, Vector3 _moveVector)
        {
            this.playerMover = _playerMover;
            this.moveVector = _moveVector;
        }

        public void Execute()
        {
            playerMover.Move(moveVector);
        }

        public void Undo()
        {
            playerMover.Move(-moveVector);
        }
    }
}
```

`PlayerMove` 클래스를 DI 받아서 Move 함수를 실행시키는 역할을 합니다.

### CommandInvoker 구현

CommandInvoker 는 Static 클래스 이고, 일종의 커맨드들을 관리하는 Manager 클래스라고 생각하시면 됩니다.

저희는 Undo, Redo 를 구현할 것이기 때문에, 각각의 MoveCommand를 Stack 에 담고, 하나씩 꺼내거나 삭제하며 로직을 구성할 것입니다.

```csharp
using System.Collections.Generic;

namespace Command
{
    public class CommandInvoker
    {
        private static Stack<ICommand> undoStack = new Stack<ICommand>();
        private static Stack<ICommand> redoStack = new Stack<ICommand>();

        public static void ExecuteCommand(ICommand _command)
        {
            _command.Execute();
            undoStack.Push(_command);

            redoStack.Clear();
        }

        public static void UndoCommand()
        {
            if (undoStack.Count > 0)
            {
                ICommand activeCommand = undoStack.Pop();
                redoStack.Push(activeCommand);
                activeCommand.Undo();
            }
        }

        public static void RedoCommand()
        {
            if (redoStack.Count > 0)
            {
                ICommand activeCommand = redoStack.Pop();
                undoStack.Push(activeCommand);
                activeCommand.Execute();
            }
        }
    }
}
```

## 로직 수행
여기까지 클래스를 만들었다면, 이제 커맨드 패턴을 완성하였습니다.

이제, Player 는 W,A,S,D 를 누를 때, 앞뒤좌우로 움직이며,  `U`키를 누르면, 행동이 한번 물려지고, `R`키룰 누르면 무른 행동을 다시 수행하게 됩니다.

이 패턴을 가지고 할 수 있는게 많습니다.

네트워크 통신을 할 때, 받은 요청들을 캡슐화 해서 커맨드로 저장을 하고, 하나씩 수행을 한다거나, 여러가지 키들을 조합하여 사용을 할 때 등 많은 곳에서 사용합니다.

그리고, 위에서 언급하였듯이 게임에서의 `다시보기`에서도 사용을 할 수 있습니다. 커맨드 패턴을 알기 전에는 `다시보기` 기능은 해당 장면들을 다 녹화해서 스트리밍 해주는 것 인줄 알았습니다. 그래서, 이 부하를 서버에서 어떻게 처리하는거지? 라는 의문이 있었는데, 사실은 커맨드 패턴을 사용하여 구현하였던 것입니다. 플레이어가 어느 시점에 입력하였던 정보를 커맨트로 저장하여, 나중에 순서대로 뿌려주기만 하면, 영상을 전부 저장할 필요 없이 `다시보기`기능을 수행할 수 있습니다. 

