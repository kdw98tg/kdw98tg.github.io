---
layout: post

title: "[Unity] Unity에서 StatePattern 구현하기"

categories:
  -  Game Engine
  
tags:
  - [Unity, DesignPattern, StatePattern]

toc: true
toc_sticky: true

published: true

date: 2024-08-19
last_modified_at: 2024-08-19
---

## StatePattern 이란?

유니티로 게임을 만들다 보면, 누군가 조종하지 않고 자기 혼자 판단하여 게임을 진행시키는 AI를 구현해야 할 때가 있습니다. 해당 AI는 어떠한 조건에 따라서 수행해야 하는 로직이 결정됩니다.

예를들어, Player가 공격 사정거리에 들어오면, 공격을 하고, 공격 사정거리 밖으로 나가게 되면 Patrol을 하는 경우가 있습니다.

방금 예시로 든 상황은 2가지의 경우밖에 없기 때문에 `if`문으로 간단하게 구현이 가능합니다.

하지만, 저희가 실제로 플레이 하는 게임은 그렇지 않습니다. 적은 Player를 쫓다가 특정공격을 하고, 어떨때는 뛰어다녀야 하고, 어떨때는 Player가 모르게 접근해야 하며, 체력이 일정 이하가 되면 광란상태에 빠지는 등 여러가지 `상태`를 가지게 됩니다.

이러한 `상태`들을 모두 `if`문으로 처리한다는 것은 거의 불가능에 가깝습니다. 이렇게 어려운것을 해낸다고 해도, 다음날 아침에 기획자가 수정을 요구한다면, 그야말로 재앙이 벌어지는 것입니다.

그렇다면 방법이 없는걸까요? 당연히 아닙니다. 우리는 이 문제를 `State Pattern`을 통해서 해결할 수 있습니다.

>State Pattern 이란,
>
>State Pattern은 객체의 상태를 캡슐화하여, 객체의 내부 상태가 변경될 때 해당 객체가 그 행동을 변경할 수 있도록 하는 패턴 입니다.
>간단하게, 복잡한 상태를 분리하여 관리하도록 해서, 추가 혹은 변경에 용이하게 대처할 수 있도록 합니다.

State Pattern은 AI 말고도 복잡한 행동을 수행해야 하는 Player Control 에도 사용합니다.

실제로 포트폴리오를 만들 때, 복잡한 행동을 수행하는 Player를 State Pattern으로 구현하였습니다. 아래 이미지는 StatePattern으로 구현한 Player의 상태도 입니다. 이걸 `if`문으로 전부 처리한다고 생각하니까 벌써 머리가 아파오는군요.

![복잡한 행동을 하는 Player의 상태도](/images/Pasted%20image%2020240819213815.png)

## StatePattern 구현

(해당 예제는 유니티코리아의 예제를 참고하여 만들었습니다.)

해당 예제는 플레이어의 `Idle`, `Walk`, `Jump` 상태를 전환하는 StateMachine 입니다. 각각의 조건은 다음과 같습니다.

![State 구성도](/images/Pasted%20image%2020240819222223.png)

해당 조건으로 각 상태들을 왔다갔다 합니다.

이 그림을 보아하니, 유니티의 어떤 기능과 많이 닮아 있습니다. 바로 Animatior Controller 입니다. 

![Unity Animator Controller](/images/Pasted%20image%2020240819222408.png)

네 맞습니다. 유니티의 Animator Controller 역시 StatePattern으로 구현되었습니다. 그렇다면 이러한 기능을 직접 구현해보도록 하겠습니다.

### 상태를 추상화 하기

우선 구현을 하려면, 객체가 가지는 상태를 추상화 해야 합니다. `IState`라는 인터페이스를 정의하여 추상화 하였습니다. `IState`인터페이스는 상태에 들어올 때, 상태를 실행할 때, 상태에서 벗어날 때 라는 함수들을 가지고 있습니다.

🗅 **<span style="color: #c03a92">Interface IState</span>**
```csharp
public interface IState
{
	public void Enter();
	public void Execute();
	public void Exit();
}
```

### 동작을 수행할 객체와 상태들을 관리한 StateMachine 만들기

`상태`를 추상화 하였으니, 이제는 구현을 할 차례 입니다. 우선, 상태를 수행해야 할 Player를 컨트롤 하는 클래스를 만듭니다. `PlayerInput`이라는 스크립트를 만들어서 Player의 Input을 관리합니다.

또한 SimplePlayerStateMachine 이라는 클래스를 만들어서, 각각의 상태를 전환하거나 제어하는 역할을 수행하게 만듭니다.

해당 PlayerController는 SimplePlayerStateMachine 이라는 클래스를 바탕으로 State들을 전환합니다. 

🗅 **<span style="color: #c03a92">Class PlayerInput</span>**
```csharp
public class PlayerInput : MonoBehaviour
{
    [Header("Controls")]
    [SerializeField] private KeyCode forward = KeyCode.W;
    [SerializeField] private KeyCode back = KeyCode.S;
    [SerializeField] private KeyCode left = KeyCode.A;
    [SerializeField] private KeyCode right = KeyCode.D;
    [SerializeField] private KeyCode jump = KeyCode.Space;

    private Vector3 inputVector = Vector3.zero;
    public Vector3 InputVector => inputVector;
    public bool IsJumping { get => isJumping; set => isJumping = value; }

    private bool isJumping = false;
    private float xInput = 0f;
    private float zInput = 0f;
    private float yInput = 0f;

    private void Update()
    {
        HandleInput();
    }

    public void HandleInput()
    {
        xInput = 0f;
        yInput = 0f;
        zInput = 0f;

        if (Input.GetKey(forward))
        {
            zInput++;
        }

        if (Input.GetKey(back))
        {
            zInput--;
        }

        if (Input.GetKey(left))
        {
            xInput--;
        }

        if (Input.GetKey(right))
        {
            xInput++;
        }

        inputVector = new Vector3(xInput, yInput, zInput);
        isJumping = Input.GetKey(jump);
    }
}
```

🗅 **<span style="color: #c03a92">Class PlayerController</span>**
```csharp
    [RequireComponent(typeof(PlayerInput), typeof(CharacterController))]

    public class PlayerController : MonoBehaviour
    {
        [SerializeField] private PlayerInput playerInput = null;
        private SimplePlayerStateMachine playerStateMachine = null;

        [Header("Movement")]
        [Tooltip("Horizontal speed")]
        [SerializeField] private float moveSpeed = 5f;
        [Tooltip("Rate of change for move speed")]
        [SerializeField] private float acceleration = 10f;
        [Tooltip("Max height to jump")]
        [SerializeField] private float jumpHeight = 1.25f;

        [Tooltip("Custom gravity for player")]
        [SerializeField] private float gravity = -15f;
        [Tooltip("Time between jumps")]
        [SerializeField] private float jumpTimeout = 0.1f;

        [SerializeField] private bool isGrounded = true;
        [SerializeField] private float groundedRadius = 0.5f;
        [SerializeField] private float groundedOffset = 0.15f;
        [SerializeField] private LayerMask groundLayers;

        private CharacterController characterController = null;
        public CharacterController CharController => characterController;

        public bool IsGrounded => isGrounded;
        public SimplePlayerStateMachine PlayerStateMachine => playerStateMachine;

        private CharacterController charController;
        private float targetSpeed;
        private float verticalVelocity;
        private float jumpCooldown;

        private void Awake()
        {
            playerInput = GetComponent<PlayerInput>();
            characterController = GetComponent<CharacterController>();

            playerStateMachine = new SimplePlayerStateMachine(this);
        }

        private void Start()
        {
            playerStateMachine.Init(playerStateMachine.IdleState);
        }

        private void Update()
        {
            //계속 실행시킬것임
            playerStateMachine.Execute();
        }

        private void LateUpdate()
        {
            CalculateVertical();
            Move();
        }

        private void Move()
        {
            Vector3 inputVector = playerInput.InputVector;

            if (inputVector == Vector3.zero)
            {
                targetSpeed = 0f;
            }

            //x,z 로만 움직이니까
            float currentHorizontalSpeed = new Vector3(characterController.velocity.x, 0.0f, characterController.velocity.z).magnitude;
            //공차
            float tolerance = 0.1f;

            //현재 속도가 목표 속도와 충분히 가깝지 않다면, 부드럽게 목표 속도로 전환함
            if (currentHorizontalSpeed < targetSpeed - tolerance || currentHorizontalSpeed > targetSpeed + tolerance)
            {
                targetSpeed = Mathf.Lerp(currentHorizontalSpeed, targetSpeed, Time.deltaTime * acceleration);
                // targetSpeed = Mathf.Round(targetSpeed * 1000f) / 1000f;// 선택 사항: 작은 부동 소수점 오류를 방지하기 위해 반올림
            }
            else
            {
                targetSpeed = moveSpeed;
            }

            characterController.Move((inputVector.normalized * targetSpeed * Time.deltaTime) + new Vector3(0f, verticalVelocity, 0f) * Time.deltaTime);
        }

        private void CalculateVertical()
        {
            if (isGrounded)
            {
                if (verticalVelocity < 0f)
                {
                    verticalVelocity = -2f;
                }
                if (playerInput.IsJumping && jumpCooldown < 0f)
                {
                    verticalVelocity = Mathf.Sqrt(jumpHeight * -2f * gravity);
                }
                if (jumpCooldown >= 0f)
                {
                    jumpCooldown -= Time.deltaTime;
                }
            }
            else
            {
                jumpCooldown = jumpTimeout;
                playerInput.IsJumping = false;
            }

            verticalVelocity += gravity * Time.deltaTime;

            // check if grounded
            Vector3 spherePosition = new Vector3(transform.position.x, transform.position.y + groundedOffset, transform.position.z);
            isGrounded = Physics.CheckSphere(spherePosition, 1.5f, groundLayers, QueryTriggerInteraction.Ignore);
        }

    }
```

🗅 **<span style="color: #c03a92">Class SimplePlayerStateMachine</span>**
```csharp
 public class SimplePlayerStateMachine
 {
     private IState currentState = null;

     private WalkState walkState = null;
     private JumpState jumpState = null;
     private IdleState idleState = null;

     public IState CurrentState { get { return currentState; } }
     public WalkState WalkState { get { return walkState; } }
     public JumpState JumpState { get { return jumpState; } }
     public IdleState IdleState { get { return idleState; } }

     public event Action<IState> stateChanged;

     public SimplePlayerStateMachine(PlayerController _player)
     {
         this.walkState = new WalkState(_player);
         this.jumpState = new JumpState(_player);
         this.idleState = new IdleState(_player);
     }

     public void Init(IState _state)
     {
         //state 바꿔줌
         currentState = _state;
         _state.Enter();

         //stateChanged?.Invoke(_state);
     }

     //다음 행동으로 전환
     public void TransitionTo(IState _nextState)
     {
         currentState.Exit();
         currentState = _nextState;
         _nextState.Enter();

         //stateChanged?.Invoke(_nextState);
     }

     public void Execute()
     {
         currentState?.Execute();
     }
 }
```

### 각각의 상태 캡슐화하고, 다음 행동으로 넘어갈 조건 구현하기

이제 Player의 행동이 정의가 되었으니, 각 `상태`들을 구현해 보겠습니다. 각 `상태`들은 `Player`를 움직일 것이기 때문에, `PlayerController`를 DI 해줍니다.

🗅 **<span style="color: #c03a92">Class IdleState</span>**
```csharp
public class IdleState: IState
{
	private PlayerController player = null;

	public IdleState(PlayerController _player)
	{
		this.player = _player;
	}

	public void Enter()
	{
		Debug.Log("IdleState Enter");
	}

	public void Execute()
	{
		if(!player.IsGrounded)
		{
			player.PlayerStateMachine.TransitionTo(player.PlayerStateMachine.JumpState);
		}
		if(Mathf.Abs(player.CharController.velocity.x) > 0.1f || Mathf.Abs(player.CharController.velocity.z) > 0.1f)
		{
			player.PlayerStateMachine.TransitionTo(player.PlayerStateMachine.WalkState);
		}
	}

	public void Exit()
	{
		Debug.Log("IdleState Exit");
	}
}
```


🗅 **<span style="color: #c03a92">Class JumpState</span>**
```csharp
public class JumpState:IState
{
	private PlayerController player = null;

	public JumpState(PlayerController _player)
	{
		this.player = _player;
	}

	public void Enter()
	{
		Debug.Log("JumpState Enter");
	}

	public void Execute()
	{
		if(player.IsGrounded)
		{
			if(Mathf.Abs(player.CharController.velocity.x) > 0.1f || Mathf.Abs(player.CharController.velocity.z) > 0.1f)
			{
				player.PlayerStateMachine.TransitionTo(player.PlayerStateMachine.IdleState);
			}
			else
			{
				player.PlayerStateMachine.TransitionTo(player.PlayerStateMachine.walkState);
			}
		}
	}
	public void Exit()
	{
		Debug.Log("JumpState Exit");
	}
}
```

🗅 **<span style="color: #c03a92">Class WalkState</span>**
```csharp
public class WalkState: IState
{
	private PlayerController player = null;

	public WalkState(PlayerController _player)
	{
		this.player = _player;
	}
	public void Enter()
	{
		Debug.Log("Walk Enter");
	}

	public void Execute()
	{
		if(!player.IsGrounded)
		{
			player.PlayerStateMachine.TransitionTo(player.PlayerStateMachine.JumpState);
		}
		if(Math.Abs(player.CharController.velocity.x) < 0.1f && Mathf.Abs(player.CharController.velocity.z) < 0.1f)
		{
			player.PlayerStateMachine.TransitionTo(player.PlayerStateMachine.IdleState);
		}
	}

	public void Exit()
	{
    Debug.Log("Walk Exit");
	}
}
```

이렇게 State 까지 구현을 완료한다면 심플하지 않은 `SimplePlayerStateMachine`이 구현되었습니다.

처음하면 구조가 복잡해보이지만, 상태가 많아질수록, 상태의 조건이 복잡할수록 빛을 바라는 구조입니다.

디자인패턴은 무조건 쓰면 좋은것이 아닌, 필요할 때 적재적소에 사용하는것이 중요한 것 같습니다. 오버 엔지니어링을 하게되면 오히려 복잡도만 높이게 되어 불편할 수 있습니다.

