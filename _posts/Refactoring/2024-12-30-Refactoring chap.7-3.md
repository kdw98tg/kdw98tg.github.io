---
title: "[Refactoring] 리팩터링 2판 Chapter.07 part 3 (캡슐화)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-12-30
last_modified_at: 2024-12-30
---

## 클래스 추출하기

클래스는 반드시 명확하게 추상화하고 소수의 주어진 역할만 처리해야 한다는 가이드라인이 있습니다. 하지만, 개발을 하다보면 한 클래스에 여러 기능들을 넣게 되고, 클래스 자체가 엄청 비대해지게 됩니다. 그러다보면, 확장성 없고 유지보수하기 힘든 클래스가 만들어지게 됩니다. 그렇기 때문에, 일부 데이터와 메서드를 따로 묶을 수 있다면 어서 분리를 하라는 신호입니다. 특정 데이터나 메서드 일부를 제거하면 어떤 일이 일어나는지 자문해 보면 판단에 도움이 됩니다.

개발 후반에는 여러가지의 서브클래스들이 생기는 문제도 생길 수 있습니다. 확장해야 할 기능이 무엇인지 봐야겠지만, 작은 일부의 기능을 확장하기 위해 서브클래스를 만들거나, 확장해야 할 기능이 무엇이냐에 따라 서브클래스를 만드는 방식도 달라진다면 클래스를 나눠야한다는 신호 입니다.

저도 어느순간부터, 클래스에 멤버변수가 너무 많아지게 되면, 이것은 지금 한 클래스가 여러가지 기능을 하고 있는게 아닌가? 라는 의심을 가지며 클래스를 분리하는 작업을 많이 하고 있습니다. 그렇게 하니 기능의 변화에도 유연하게 대처하고, 의존성관리도 쉬워지고, 해당 클래스를 그냥 다른 프로젝트에 복붙해도 바로 사용할 수 있기 때문에 굉장히 개발이 수월해 졌습니다.

### 절차
1. 클래스의 역할을 분리할 방법을 정한다.
2. 분리될 역할을 담당할 클래스를 새로 만든다.
3. 원래 클래스의 생성자에서 새로운 클래스의 인스턴스를 생성하여 필드에 저장해둔다.
4. 분리될 역할에 필요한 필드들을 새 클래스로 옮긴다. 하나씩 옮길때마다 테스트한다.
5. 메서드들도 새 클래스로 옮긴다. 이때, 저수준 메서드, 즉 다른 메서드를 호출하기보다는 호출을 당하는 일이 많은 메서드부터 옮긴다. 하나씩 옮길 때마다 테스트한다.
6. 양쪽 클래스의 인터페이스를 살펴보면서 불필요한 메서드를 제거하고, 이름도 새로운 환경에 맞게 바꾼다.
7. 새 클래스를 외부로 노출할지 정한다. 노출하려거든 새 클래스에 참조를 값으로 바꾸기를 적용할지 고민해본다.

### 예제

해당 예제는 책에 있는 코드가 잘 와닿지 않아 Unity 엔진의 예제를 가져왔습니다. 해당 예제는 Player 스크립트에서 움직임을 구현하는 코드 입니다.

```csharp
public class Player : MonoBehaviour
{
    //움직임에 필요한 변수들 ..
    private float moveSpeed = 0f;
    private float moveDamping = 0f;

    //입력관련 변수들
    private bool isMoveLeft;
    private bool isMoveRight;
    private bool isMoveForward;
    private bool isMoveBack;

    private void Update()
    {
        HandleInput();
        Move();
    }
    private void HandleInput()
    {
        isMoveLeft = Input.GetKey(KeyCode.A);
        isMoveRight = Input.GetKey(KeyCode.D);
        isMoveForward = Input.GetKey(KeyCode.W);
        isMoveBack = Input.GetKey(KeyCode.S);
    }
    private void Move()
    {
        if (Input.GetKeyDown(KeyCode.A))
        {
            MoveLeft();
        }
        if (Input.GetKeyDown(KeyCode.S))
        {
            MoveBack();
        }
        if (Input.GetKeyDown(KeyCode.D))
        {
            MoveRight();
        }
        if (Input.GetKeyDown(KeyCode.W))
        {
            MoveForward();
        }
    }
    private void MoveLeft()
    {
        //왼쪽으로 움지기는 로직
    }
    private void MoveRight()
    {
        //오른쪽으로 움직이는 로직
    }
    private void MoveForward()
    {
        //앞쪽으로 움직이는 로직
    }
    private void MoveBack()
    {
        //뒤로 움직이는 로직
    }
}
```

해당 코드를 보면, 움직임을 나타내는 변수들과 함수들, 입력을 처리하는 함수들과 변수들이 한 스크립트에 있어 클래스가 비대해 집니다. 이 스크립트를 기능별로 나누게 된다면, 플레이어의 움직임을 제어하는 `PlayerMovement` 클래스와 플레이어의 입력을 처리하는 `PlayerInput`로 나눌 수 있습니다.

```csharp
public class Player : MonoBehaviour
{
    private PlayerMovement playerMovement = null;
    private PlayerInput playerInput = null;

    private void Awake()
    {
        playerMovement = GetComponent<PlayerMovement>();
        playerInput = GetComponent<PlayerInput>();
    }

    private void Awake()
    {
        MovePlayer();
    }

    private void MovePlayer()
    {
        if (playerInput.GetIsMoveBack())
        {
            playerMovement.MoveBack();
        }
        if (playerInput.GetIsMoveForward())
        {
            playerMovement.MoveForward();
        }
        if (playerInput.GetIsMoveLeft())
        {
            playerMovement.MoveLeft();
        }
        if (playerInput.GetIsMoveRight())
        {
            playerMovement.MoveForward();
        }
    }
}
```

```csharp
public class PlayerMovement : MonoBehaviour
{
    //움직임에 필요한 변수들 ..
    private float moveSpeed = 0f;
    private float moveDamping = 0f;

    public void MoveLeft()
    {
        //왼쪽으로 움지기는 로직
    }
    public void MoveRight()
    {
        //오른쪽으로 움직이는 로직
    }
    public void MoveForward()
    {
        //앞쪽으로 움직이는 로직
    }
    public void MoveBack()
    {
        //뒤로 움직이는 로직
    }
}
```

```csharp
public class PlayerInput : MonoBehaviour
{
    //입력관련 변수들
    private bool isMoveLeft;
    private bool isMoveRight;
    private bool isMoveForward;
    private bool isMoveBack;

    private void Update()
    {
        HandleInput();
    }

    private void HandleInput()
    {
        isMoveLeft = Input.GetKey(KeyCode.A);
        isMoveRight = Input.GetKey(KeyCode.D);
        isMoveForward = Input.GetKey(KeyCode.W);
        isMoveBack = Input.GetKey(KeyCode.S);
    }

    public bool GetIsMoveLeft()
    {
        return isMoveLeft;
    }
    public bool GetIsMoveRight()
    {
        return isMoveRight;
    }
    public bool GetIsMoveForward()
    {
        return isMoveForward;
    }
    public bool GetIsMoveBack()
    {
        return isMoveBack;
    }
}
```

위와 같이 리팩터링하게 되면, 클래스가 비대해지는것을 막을 수 있고, 역할이 딱 나눠져 있어서 유지보수하기도 쉽고, 재사용하기도 쉬운 코드로 바뀌게 됩니다.

## 클래스 인라인 하기

클래스 인라인 하기의 개념은 위에서 살펴보았던 클래스 추출하기와는 반대되는 개념 입니다. 제 역할을 못하여 그대로 두면 안되는 클래스는 인라인하는 것입니다. 클래스를 추출하다가보니 역할이 거의 없는 빈껍데기 클래스가 남게되면 클래스 인라인하기로 다시 합쳐줍니다.

### 절차
1. 소스 클래스의 각 public 메서드에 대응하는 메서드들을 타깃 클래스에 생성한다. 이 메서드들은 단순히 작업을 소스 클래스로 위임해야 한다.
2. 소스 클래스의 메서드를 사용하는 코드를 모두 타깃 클래스의 위임 메서드를 사용하도록 바꾼다. 하나씩 바꿀 때마다 테스트한다.
3. 소스 클래스의 메서드와 필드를 모두 타깃 클래스로 옮긴다. 하나씩 옮길 때마다 테스트한다.
4. 소스클래스를 단칼에 삭제 시킨다.

### 예제
Person클래스 안에 전화번호에 관한 정보들이 있다고 가정 합니다. 전화번호에 관한 정보가 예전에는 비대해서 하나의 클래스로 묶어서 처리하였다고 가정 합니다.

```js
class Person {
    get officeAreaCode() {
        return this._officeAreaCode;
    }
    set officeAreaCode(arg) {
        this._officeAreaCode = arg;
    }
}

class TelephoneNumber {
    get officeAreaCode() {
        return this._officeAreaCode;
    }
    set officeAreaCode(arg) {
        this._officeAreaCode = arg;
    }
}
```

하지만, 지금 보면 Person 클래스에서 단순히 TelephoneNumber의 필드를 반환하고 있기 때문에, 해당 클래스는 필요없어 보입니다. 따라서 TelephoneNumber 클래스의 필드들을 Person 클래스로 인라인 해줍니다.

```js
class Person {
    constructor(officeAreaCode, officeNumber) {
        this._officeAreaCode = officeAreaCode;
        this._officeNumber = officeNumber;
    }
    get officeAreaCode() {
        return this._officeAreaCode;
    }
    set officeAreaCode(arg) {
        this._officeAreaCode = arg;
    }
    get officeNumber() {
        return this._officeNumber;
    }
    set officeNumber(arg) {
        this._officeNumber = arg;
    }
}
```


## 위임 숨기기

모듈과 설계를 제대로 하는 핵심은 캡슐화 입니다. 처음에 캡슐화를 접했을 때, public 필드와 뭐가 다르지? 라고 생각을 하였습니다. 지금와서는 캡슐화의 진가를 알게 되었고 캡슐화 없이는 코드를 짤 수 없는 지경에 이르렀습니다. 

이 이유는 public 필드와 getter 역할을 하는 캡슐화된 함수의 차이는 확장성입니다. public 필드만을 가져오게 되면 public 필드를 참조하는 쪽에서 해당 필드를 가지고 연산을 수행해야 합니다. 히지만 캡슐화가 되어 있다면, 연산이 끝난 결과문을 반환하기 때문에 호출할 때 마다 같은 로직을 사용할 필요가 없습니다. 이는 유지보수에까지 이어집니다. 만약 캡슐화가 되어있지 않고 클라이언트에서 호출할 때 마다 로직을 짜게 만들었다면, 호출한 클라이언트들을 모두 기억한 다음, 해당 클라이언트에 가서 정보를 수정해야 합니다.

이것은 서버와 클라이언트에도 똑같이 적용 됩니다. 클라이언트가 서버의 위임객체(클래스 안에 있는 객체)를 호출하려면 위임 객체가 무엇인지 알아야 합니다. 이러한 의존성을 없애기 위해서 서버 자체에 위임 메서드를 만들어서 위임 객체를 숨기는 식으로 코딩해야 합니다. 예제로 살펴 보겠습니다.

### 절차
1. 위임 객체의 각 메서드에 해당하는 위임 메서드를 서버에 생성한다.
2. 클라이언트가 위임 객체 대신 서버를 호출하도록 수정한다. 하나씩 바꿀 때마다 테스트한다.
3. 모두 수정했다면, 서버로부터 위임 객체를 얻는 접근자를 제거한다.

### 예시

사람과 사람이 속한 부서를 정의 합니다. 사람 클래스 안에 위임객체인 부서가 있다고 가정합니다.

```js
class Person {
    constructor(name) {
        this._name = name;
    }
    get name() {
        return this._name;
    }
    get department() {
        return this._department;
    }
    set department(arg) {
        this._department = arg;
    }
}

class Department {
    constructor(chargeCode, manager) {
        this._chargeCode = chargeCode;
        this._manager = manager;
    }
    get chargeCode() {
        return this._chargeCode;
    }
    set chargeCode(arg) {
        this._chargeCode = arg;
    }
}
```

이렇게 정의가 되어 있다고 하면, Person 에서 Department 를 가져오려면 클라이언트에서는 다음과 같은 코드를 짜야 합니다.

```js
Person aPerson = new Person("Kim");
manager = aPerson.department.manager;
```

이렇게 되면, Person 객체의 `manager`를 가져오기 위해서는 위임 객체인 `Department`까지 알고 있어야 합니다. 이러한 의존성을 떼기 위해서 다음과 같이 수정합니다.

```js
class Person {
    constructor(name) {
        this._name = name;
    }
    get name() {
        return this._name;
    }
    get manager() {
        return this._department.manager;
    }
}
```

getter에 위임 객체인 department를 반환하는것이 아닌 department의 manager를 바로 반환하도록 합니다. 이렇게 하면, 클라이언트는 부서 클래스의 작동 방식, 다시 말해서 부서 클래스가 관리자 정보를 제공한다는 사실을 몰라도 됩니다.