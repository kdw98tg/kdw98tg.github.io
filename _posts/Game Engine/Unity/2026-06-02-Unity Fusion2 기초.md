---
layout: post

title: "[Unity] Unity Fusion2 기초"

categories:
  -  Game Engine
  
tags:
  - [Unity, Fusion2, Photon]

toc: true
toc_sticky: true

published: true

date: 2026-06-07
last_modified_at: 2025-06-07
---

## NetworRunner

Fusion2를 사용할 때, 네트워크 기능들을 모두 관리하는 객체임. Fusion2의 핵심 컴포넌트이고, Tick 시뮬레이션, 서버 연결, 콜백 수신 등 모두 담당하고 있습니다.

NetworkBehaviour를 상속받는 객체들은 생성된 Runner를 바로 참조할 수 있는 프로퍼티를 제공합니다.
## NetworkBehaviour 

네트워크 오브젝트의 기반임. Unity의 MonoBehaviour 대신 상속함. 네트워크 상에 존재하는 오브젝트에게 붙입니다.

Spawned() Despawned() 라이프사이클을 제공 합니다. (MonoBehaviour의 Start() 와 Destroy()와 같음)

네트워크 기능이 필요한 객체에만 상속 시켜주면 됩니다.

## Networked 어트리뷰트

```cs
[Networked, OnChangedRender(nameof(OnHpUpdated))]
public int HP { get; private set; } = 3;

 public void OnHpUpdated()
    {
        if (HasInputAuthority)//Hp는 깎이면, 내 입력 권한 에서만 업데이트
        {
            CommandInvoker<UpdatePlayerHpUIParam>.Execute(new UpdatePlayerHpUIParam()
            {
                CurrentHp = HP
            });
        }
        //소리는 같이 재생
        CommandInvoker<PlaySfxWithPoolParam>.Execute(new PlaySfxWithPoolParam()
        {
            SFX = SFXS.ShootGun,
            Enable3DSound = false,
        });
    }
```

네트워크에서 모든 유저들에게 동기화가 되어야 하는 아주 중요한 변수들인 경우, `[Networked]`라는 어트리뷰트를 사용함. OnChangedRender 라는 속성에는 Hp가 바뀌었을때, 작동되어야 하는 함수를 넣어주면 됩니다.

## RPC (Remote Procedure Call)

RPC는 서버(호스트)에 함수를 저장하여, 서버든 클라이언트든 서버에 있는 함수를 사용하여 특정 동작을 수행하거나 동기화 합니다.

자주 쓰는 3가지 패턴이 있습니다.

```cs
// 패턴 1: 클라이언트 → 서버 (값 보고)
[Rpc(RpcSources.InputAuthority, RpcTargets.StateAuthority)]
public void RPC_SendBeatsToServer(float[] _beatTimes, RpcInfo info = default)

// 패턴 2: 서버 → 전체 (연출 동기화)
[Rpc(RpcSources.StateAuthority, RpcTargets.All)]
public void RPC_ChangeBattelState(BattleStateId _id)

// 패턴 3: 서버 → 특정 클라이언트 (개인 데이터 전달)
[Rpc(RpcSources.StateAuthority, RpcTargets.InputAuthority)]
public void RPC_SetOpponentBeat(float[] _shuffledBeats)
```

`RpcSources.InputAuthority`는 입력 권한만 가진 주체를 의미함. 즉, 클라이언트를 뜻함. 반대로 `RpcTargets.StateAuthority`는 상태를 바꿀 수 있는 권한을 가진 주체를 의미함. 즉, 서버(호스트)를 뜻합니다.

RPC의 인자로는 첫번쨰로, `RpcSources` 라는 주체에 관한 권한과, 두번째로, `RpcTargets`라는 대상에 대한 권한으로 구성됩니다.
## NetworkInput

또한, 퓨전에서는 해킹을 방지하고 입력 오차를 줄이기 위해 클라이언트의 키보드/마우스 입력을 직접 처리하지 않고 구조체로 묶어서 서버로 보냅니다.

이전에 구현했을때는, 삐 소리를 서버에서 클라이언트쪽으로 전파하면, 클라이언트는 자기가 받은 시간을 기록하고, 클라이언트에서 인풋을 눌렀을 때, 오차를 계산하여 서버로 보내는 방식을 사용했었음 (클라이언트 신뢰 방식이라고 하는데 클라이언트 신뢰 방식은 네트워크에서 지양해야하는 방식인듯)

근데 그거보다 NetworkInput을 사용해서 누를 시점의 Tick을 서버로 같이 보내고 서버에서 판정하는게 더 좋은 방식이라고 합니다.

```cs
// 1. 주고받을 입력 데이터 정의 (주로 Common 폴더에서 관리)
public struct NetworkInputData : INetworkInput
{
    public bool isSpacePressed;
}

// 2. 플레이어 스크립트에서의 사용 (FUN)
public override void FixedUpdateNetwork()
{
    // 권한을 가진 유저의 입력 데이터를 틱 단위로 정확하게 받아옴
    if (GetInput(out NetworkInputData data))
    {
        if (data.isSpacePressed)
        {
            // 스페이스바 입력 처리 로직 (이동, 발사 등)
        }
    }
}
```

## NetworkPrefabRef

네트워크 상에서 플레이어나 총알, 아이템 등을 생성할 때는 유니티의 기본 함수인 `Instantiate`를 사용하면 안됨. 반드시 NetworkRunner의 `Spawn()` 함수를 사용해야 합니다.

그리고, Spawn 함수는 GameObject를 받는게 아닌, NetworkObject 라는 객체가 붙어있는 프리팹을 받게 됨. 그때, 사용할 수 있는것이 `NetworkPrefabRef`입니다.

## PlayerRef

네트워크 방(세션)에 접속한 유저들에게 부여되는 고유 식별 번호 임

`OnPlayerJoined` 이벤트에서 네트워크에 접속한 플레이어들 (네트워크 객체 생성 전)에게 고유한 번호를 부여함. 그러면 `OnPlayerJoined`의 인자로 Fusion2가 부여한 `PlayerRef`를 달아주게 됨.

그렇게 `Spawn()`함수로 생성을 했다면, 플레이어를 하나 선택할때는 `PlayerRef` 를 가지고 선택을 하게 됩니다.

## 권한 (Object.HasStateAuthority, Object.HasInputAuthority)

멀티플레이 게임에서는 이 과정을 서버와 클라이언트의 역할로 나눠야 합니다. 모든 네트워크 통신이 그렇듯, 클라이언트가 중요한 임무를 맡아서 수행하는것을 권장하지 않습니다. 왜냐하면, 클라이언트에게 상태 변경과 같은 중요한 임무를 맏기게 되면, 클라이언트가 데이터를 직접 조작하여 재화를 늘린다던지, 무적으로 만든다던지의 부정한 행동을 할 가능성이 높아지기 때문입니다. 따라서, 서버에서 각 클라이언트의 상태를 변경할 수 있는 권한을 가지고 클라이언트의 상태가 변경되어야 할 때, 서버가 클라이언트의 상태를 바꿔줘야 합니다. 클라이언트는 단순히 입력만을 서버로 보내는 역할만을 담당하는 것이죠.

그렇다면, 지금 이 유저가 **상태르 변경할 수 있는 권한이 있는가?**, **입력에 대한 권한만을 가지고 있는가?** 를 판정해야 합니다. Photon Fusion2에서는 이러한 조건을 검사할 수 있도록 프로퍼티를 제공합니다. 해당 프로퍼티는 `NetworkBehaviour`를 상속받은 객체들만 사용할 수 있습니다. 권한의 종류는 다음과 같습니다.

### HasStateAuthority

HasStateAuthority는 상태를 바꿀 수 있는 권한이 있는가? 에 대한 bool 형 변수 입니다. 해당 자격은 호스트 또는 서버가 가지게 됩니다. 즉, 호스트 또는 서버는 네트워크상의 모든 오브젝트의 상태를 변경할 수 있는 권한을 가지고 있는 겁니다. 따라서 게임에 영향이 가는 상태들을 바꾸거나 조작할때는 해당 권한이 있는지 체크하는 코드가 필요합니다.

```cs
[Networked] public float Hp{get;set;}

private void GetDamage(float _damage)
{
	if(Object.HasStateAuthoirty)
	{
		//체력을 변경하는 로직
		Hp -= _damage;
	}
}
```

위 코드처럼 권한을 체크하게 되면, 클라이언트에서는 아무리 체력을 바꾸려고해도 바꿀 수 없게 됩니다.

여기서 헷갈렸던 점이, Fusion2의 `NetworkRunner.IsServer()` 도 결국 서버 또는 호스트인지 구별하는 코드인데, `Object.HasInputAuthority`와 어떻게 다른가? 였습니다.

이것을 굳이 나눈 이유는 네트워크 방식이 `Host/Client`방식만이 있는게 아니라 중앙 서버에 모든 유저들이 들어가서 게임을 즐기는 `Shared`방식이 있기 때문입니다.

`Host/Client`방식에서는 Server와 상태를 변경할 수 있는 권한을 가진 유저가 동일하기 때문에 문제가 되지 않지만, `Shared`모드에서는 달라질 수 있게 됩니다. 그러면, `Host/Client`방식에서 `Shared`모드로 방식을 바꾸게 되면, 불필요하게 코드를 변경해야 하는 경우가 발생하게 됩니다.

`Shared` 모드에서는 모든 유저가 `Client`이기 때문에, `NetworkRunner.IsServer`로 체크하면 다 False가 떠서 동기화가 이루어지지 않습니다.

이런 불편함을 없애기 위해서 `NetworkRunner.IsServer`와 `Object.HasStateAuthority`를 나누었고, 네트워크 방식이 바뀌어도 코드를 거의 수정하지 않을 수 있게 됩니다.

정리하자면, 사용처는 다음과 같습니다.

Object.HasStateAuthority
- 네트워크 상의 스폰된 특정 사물(Object)의 데이터나 상태를 변경할 때 사용함
- 직접 개발하는 인게임 플레이 로직(이동, 전투, 상태 전환 등)95% 이상은 무조건 이걸 쓴다고 생각하면 됨
- State전환
- 체력 데미지 처리
- 드롭된 파츠나 플레이어의 Transform 위치
- 해당 오브젝트에 붙어있는 TickTimer를 쓸때

NetworkRunner.IsServer
- 특정 사물이 아니라, 게임방, 전체의 환경이나 글로벌 룰을 제어할 때 사용함
- 어느 하나의 캐릭터에 속한 행동이 아니라면 이걸 씀
- 오브젝트 최초 생성
- 씬 이동
- 유저 세션 관리
- 매치메이킹 룰

### HasInputAuthority

`HasInputAuthority`권한은 오브젝트에 내 키보드/마우스 입력 데이터를 넣을 수 있는 권한입니다. 10명의 유저가 있다고 하면, 10명의 유저 모두 자기의 캐릭터 1개에 대해서만 InputAuthority를 가지게 됩니다. 예를들어, 오직 내 캐릭터를 통해서만 스페이스가 눌러졌다고 서버에 알릴 수 있는것입니다.


## FixedUpdateNetwork (FUN)

네트워크 프로그래밍에서 가장 어려운 점을 꼽으라면 단연 '모든 클라이언트에서의 동기화(Synchronization)'일 것입니다.

플레이어들은 각자 사양이 다른 컴퓨터를 사용하고, 네트워크 환경(지연 시간, 패킷 손실 등) 또한 천차만별입니다. 따라서 A 클라이언트와 B 클라이언트가 완전히 동일한 타이밍에 조작을 하더라도, 두 화면에서 반드시 같은 시간에 같은 결과물이 나타날 것이라는 보장이 없습니다.

유니티의 `MonoBehaviour`에서 제공하는 `Update`는 각자의 로컬 환경에 따라서 매 프레임 돌아가는  함수이기 때문에, 해당 문제를 해결할 수 없습니다. 즉, 위 동기화 문제를 해결하려면, 하나의 `Update`에서 로직을 처리해야 합니다.

Fusion2에서는 이러한 문제를 해결하기 위해 단 하나의 `Update`인 `NetworkBehaviour.FixedUpdateNetwork` 일명 `FUN`이라는 함수를 제공합니다. 

`FUN`은 다음과 같은 특징이 있습니다.

### FUN의 특징

1. 틱(Tick) 기반의 시뮬레이션

Fusion은 가변적인 렌더링 프레임 대신, 네트워크 상에서 약속된 고정 시간 간격인 틱(Tick)을 기준으로 게임의 상태를 시뮬레이션합니다. `FixedUpdateNetwork()`는 이 틱마다 호출되므로, 기기 사양이나 네트워크 지연과 무관하게 모든 클라이언트가 **동일한 횟수, 동일한 시간 간격**으로 로직을 실행하도록 보장합니다.

2. 재시뮬레이션(Resimulation)과 예측(Prediction)

네트워크 지연(Lag)으로 인해 서버의 데이터가 늦게 도착하더라도 게임이 끊기지 않습니다. Fusion은 로컬 클라이언트의 입력을 바탕으로 먼저 예측(Prediction)하여 `FixedUpdateNetwork()`를 실행합니다. 이후 서버로부터 정확한 과거 상태를 전달받았을 때 오차가 있다면, 틀린 부분을 롤백한 뒤 순식간에 재시뮬레이션(Resimulation)하여 상태를 완벽하게 동기화합니다.

3. 호스트에서만 실행
FUN은 1개만 동작해야 문제가 발생하지 않으므로, 해당 함수는 `Server/Host`에서만 호출이 됩니다.

### FUN을 사용해야 하는 로직

그렇다면, 모든 로직을 FUN에서 처리해야 하는것일까요? 유니티를 조금 다뤄봤다면, 하나의 `Update`에서 모든 로직을 처리하려고 하면, bool 값도 너무 많아지고 코드도 엄청 길어지게 됩니다. 또한, Fusion2는 Coroutine 이나 UniTask 같은 비동기 로직도 `FUN`에서 다 처리하도록 되어 있습니다.

동기화가 중요한 네트워크에서도 마찬가지 입니다. 모든 UI 처리, 이펙트 처리, 사운드 처리를 `FUN`에서 하려고 하면, 코드도 너무 길어지고 불필요하게 네트워크 관련 로직들이 무거워질 수 있습니다.

따라서, `FUN`을 사용해야하는 곳은 **게임에 지대한 영향을 미칠 수 있는 로직**에만 사용하면 됩니다. 즉, 게임에 영향을 미치는 게임 판정, 상태 변경 등등은 `FUN`을 쓰면 되고 카운트다운을 해주는 UI를 1초마다 표시하는 기능 등과 같이 조금 엉성해도 게임에는 지장이 없는 로직들은 각 클라이언트에서 실행할 수 있도록 `Coroutine`또는 `UniTask`를 사용하면 됩니다.

네트워크에서 가장 중요한 것은 게임의 판정(논리)와 연출(화면)을 철저히 분리 하는것 이라고 할 수 있겠습니다.

다음 예제는 모든 플레이어가 3초뒤 게임 시작 이라는 로그를 보고, 실제로 3초뒤에 모든 플레이어의 상태를 전환하는 예제 입니다. 여기서, 모든 플레이어가 3초뒤 게임 시작 이라는 로그를 보는 것은 게임에 영향을 미치지 않으므로, `FUN`에 넣지 않아도 되고, 모든 플레이어의 상태를 전환하는것은 게임에 지대한 영향(다음 상태로 넘어가지 못한 플레이어는 게임을 플레이할 수 없음)을 미치기 때문에 `FUN`에서 관리하도록 설계하였습니다.

또한, 상태를 전환하는 것은 모든 클라이언트들이 하는것이 아니고, Host가 관리해야하기 때문에 위에서 다룬 `Object.HasInputAuthority`를 사용해서 조건처리를 하였습니다.

(해당 예제의 OnFixedUpdateNetwork 는 NetworkBehaviour를 상속받은 StateMachine이 State Interface와 소통할 수 있도록 만든 커스텀 함수 입니다. `FUN`은 StateMachine에서 돌아가고 있습니다.)

```cs
using Cysharp.Threading.Tasks;
using Fusion;
using Unity.VisualScripting;

public class ReactionSpeedBattleState_RoundStart : IReactionSpeedBattleState
{
    private readonly ReactionSpeedStateMachine stateMachine = null;
    private readonly PlayerManager playerManager = null;

    private const float WAIT_TIME = 3f;
    private float shouldStartTime = 0f;

    public ReactionSpeedBattleState_RoundStart(ReactionSpeedStateMachine _stateMachine, PlayerManager _playerManager)
    {
        stateMachine = _stateMachine;
        playerManager = _playerManager;
    }

    public void Enter()
    {
        Logger.Log("라운드 시작");
        Logger.Log("3초 뒤 게임 시작");

        // 서버만 게임 로직 실행, 클라이언트는 [OnChangedRender]로 상태 전환을 받음
        if (stateMachine.Object.HasStateAuthority)
        {
            playerManager.SpawnNetworkPlayer();
        }

        shouldStartTime = stateMachine.Runner.SimulationTime + WAIT_TIME;

        CountDown().Forget();
    }

    public void Exit()
    {
        Logger.Log("라운드 시작 상태 종료");
    }

    public void OnFixedUpdateNetwork()
    {
        if (!stateMachine.Object.HasStateAuthority)
        {
            return;
        }

        float currentTime = stateMachine.Runner.SimulationTime;

        if (currentTime >= shouldStartTime + 2f)
        {
            stateMachine.OnChangeNetworkStateType(ReactionBattleStateType.DropParts);
        }
    }

    private async UniTaskVoid CountDown()
    {
        Logger.Log("3초뒤 게임 시작함");

        for (int i = (int)WAIT_TIME; i > 0; i--)
        {
            Logger.Log($"{i}...");
            await UniTask.Delay(1000); // 1초 대기
        }

        Logger.Log("시작!");
    }
}
```

### Tick 관련 처리

`FUN`에서는 하나의 통일된 시간을 Tick을 통해서 전달합니다. 해당 Tick을 사용할 수 있는 프로퍼티들을 소개합니다.
#### NetworkRunner.SimulationTime

해당 프로퍼티는 네트워크 게임이 시작되고 나서 Tick이 돌면서 얼마나 시간이 지났는지를 확인할 수 있게 해줍니다. 즉, 모든 컴퓨터가 공유하는 절대적인 서버 시간 입니다. Time.deltaTime이나 Time.time은 내 컴퓨터의 성능에 따라서 늘어나거나 줄어들 수 있기 때문에 네트워크에서는 사용이 금기시 됩니다. 반면, `SimulationTime`은 퓨전의 틱을 기반으로 계산되는 시간이라 안전합니다.(초당 60회 고정임) 간단한 로직으로는 3초뒤 상태 변경과 같은 로직들이 있습니다.

```cs
 public void OnFixedUpdateNetwork()
    {
        if (!stateMachine.Object.HasStateAuthority)
        {
            return;
        }

        float currentTime = stateMachine.Runner.SimulationTime;

        if (currentTime >= shouldStartTime + 2f)
        {
            stateMachine.OnChangeNetworkStateType(ReactionBattleStateType.DropParts);
        }
    }
```

#### NetworkRunner.IsResimulation

Fusion2 엔진에서는 특정 클라이언트에서 알 수 없는 이유로 렉이 걸려서 동기화가 깨질 경우, 그 간극을 고치려고 과거로 돌아가서 다시 계산을 하게 됩니다. 해당 상태를 `Resimulation`이라고 합니다. 예를들어, 멀티 FPS 게임에서 내가 총을 쐈는데, 총을 쏘기 전에, 사실 총에 맞아서 죽었는데 그 정보를 담은 패킷이 늦게 도착했다고 가정해 봅시다. 이때, Fusion2 엔진은 게임을 과거의 틱으로 되감은 후, 현재까지의 코드를 1프레임 안에 초고속으로 다시 실행하여 결과를 바로잡습니다. 이때, `Resimulation`이 true가 됩니다.

여기까지는 문제가 없어 보이지만, 만약, 연산과정에서 `Sound`를 출력하는 코드나, `Effect`를 터트리는 코드가 있다라고 가정하면, 문제가 생길 수 있습니다.

다시 과거로 돌아가서 연산을 하는 과정에 `Sound`또는 `Audio`가 여러번 출력될 수 있기 때문입니다.

따라서, 사운드 재생, 파티클 생성, 로컬 UI 팝업 등 시각적 또는 청각적 연출을 할때는 반드시 해당 변수가 false 일때만 실행하도록 해야 합니다.

```cs
 public void OnFixedUpdateNetwork()
    {
        float currentTime = stateMachine.Runner.SimulationTime;

        if (!localBeepPlayed && currentTime >= stateMachine.BeepTime)
        {
            localBeepPlayed = true;

            if (!stateMachine.Runner.IsResimulation)
            {
                AudioManager.Instance.PlaySfx(SFXS.Beep);
                Logger.Log("삐! 스페이스를 눌러라!");
            }
        }
    }
```
