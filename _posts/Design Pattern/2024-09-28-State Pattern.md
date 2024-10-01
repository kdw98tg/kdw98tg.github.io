---
title: "[Design Pattern] 상태 패턴 (State Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2024-09-28
last_modified_at: 2024-09-28
---

## State Pattern 이란?

> 상태 패턴은 특정 상태에 따라 다른 행위를 하는 상황에서 상태를 조건문으로 나누는 것 대신, 상태를 객체화 하여 상태가 행동을 할 수 있도록 위임하는 패턴을 말합니다.
> 
> 이 패턴의 장점은, 상태를 객체화 해서, 상태를 추가하거나, 상태를 수정하는데 용이하다는 것이 있습니다. 물론 이 과정에서 다른 상태에 영향을 주지 않습니다.


### 패턴의 사용 시기

- 객체가 상태에 따라 다른 행동을 할 때
- 상태 및 전환에 대규모 조건문이 걸려있을 경우
- 런타임에서 상태가 유동적으로 바뀌어야 하는 경우

### 패턴의 장점

- 복잡한 상태를 캡슐화해서 개별 클래스로 관리할 수 있음
- 상태에 대한 내용을 클래스로 관리하면서, 복잡도를  낮춤
- 단일책임원칙, 개방폐쇄원칙 을 준수할 수 있음

### 패턴의 단점

- 상태마다의 클래스가 생기기 때문에, 관리해야 할 클래스가 늘어남
- 객체의 상태가 많이 없거나, 상태변환이 자주 일어나지 않는경우에는 복잡도만 증가함

## 예제

State Pattern 은 상태의 변환에 유동적으로 대처하기 위해 나온 패턴입니다.

이번에 다뤄볼 예제는 껌볼 머신을 상태별로 구분해서, 껌볼을 뽑는 기계를 만들어 봅니다.

상태의 흐름도는 다음과 같습니다.

![](/images/Pasted%20image%2020240928185145.png)

일단 어떤 조건에 따라서 상태가 분기 되기 때문에, if 문 또는 switch ~ case 문으로 해당 알고리즘을 작성하게 되면, 처음 만들때는 쉬울 수 있으나, 상태가 추가되거나, 조건이 변경될 때 변경에 용이하지 않습니다. 또한, 운좋게 변경했다 하더라도, 시간이 매우 많이 걸리고, 다른사람이 코드를 보게 되면 한숨을 자아내게 됩니다.

이러한 문제를 해결하기 위해, `State Pattern`을 가지고 알고리즘을 구현 합니다.

### 상태 Interface 작성

우선, `상태`라는 공통적인 인터페이스를 만듭니다.

이번에 구현하는 예제는 `동전을 넣는다`, `동전을 반환한다`, `손잡이를 돌린다`, `껌볼이 나온다`, `껌볼을 채운다` 의 행동을 합니다.\

🗅 **<span style="color: #c03a92">interface State</span>**

```java
package src.state.gumballstate;  
  
public interface State {  
	//동전을 넣는다
    void insertQuater();  
    //동전을 반환함
    void ejectQuater();  
    //손잡이를 돌린다
    void turnCrank();  
    //껌볼이 나온다
    void dispense();  
    //껌볼을 다시 채운다
    public void refill();  
}
```

### 상태머신 만들기

이제 이 상태들을 제어할 `State Machine`을 제작 합니다. 해당 상태머신은 모든 상태들을 다 가지고 있고, 상태들을 전환하거나, 상태를 반환하는 역할을 합니다.

객체의 상태는 위의 다이어그램에서 정의 했듯, `동전이 있는 상태`, `알맹이를 판매한 상태`, `동전이 없는 상태`, `알맹이가 매진된 상태`로 정의 합니다.


우선 `StateMachine`을 만들고, 각 상태들을 만들도록 하겠습니다.

🗅 **<span style="color: #c03a92">class GumballMachine</span>**

```java
package src.state.gumballstate;  
  
public class GumballMachine {  
	//상태들 선언
    private State soldState;  
    private State soldOutState;  
    private State noQuarterState;  
    private State hasQuarterState;  

	//현재 상태 선언
    private State state;  
  
    int count = 0;  
  
    public GumballMachine(int numberGumballs) {  
        soldOutState = new SoldOutState(this);  
        noQuarterState = new NoQuaterState(this);  
        hasQuarterState = new HasQuaterState(this);  
        soldState = new SoldState(this);  
  
        this.count = numberGumballs;  
        if (numberGumballs > 0) {  
            state = noQuarterState;  
        } else {  
            state = soldOutState;  
        }  
    }  
  
    public void insertQuater(){  
        state.insertQuater();  
    }  
  
    //꺼내다  
    public void ejectQuater(){  
        state.ejectQuater();  
    }  
  
    //돌리다  
    public void turnCrank(){  
        state.turnCrank();  
        state.dispense();//분배하다  
    }  
  
    //나오다  
    public void releaseBall() {  
        System.out.println("슬롯에서 알맹이가 나오는 중입니다.");  
        if (count != 0) {  
            count = count - 1;  
        }  
    }  
  
    public void refill(int _count){  
        this.count += _count;  
        System.out.println("껌볼이 리필됨. 현재 검볼 : "+ _count);  
        state.refill();  
    }  
  
    public int getCount(){  
        return count;  
    }  
    public State getState(){  
        return state;  
    }  
    public void setState(State state){  
        this.state = state;  
    }  
    public State getSoldState(){  
        return soldState;  
    }  
    public State getSoldOutState(){  
        return soldOutState;  
    }  
    public State getNoQuarterState(){  
        return noQuarterState;  
    }  
    public State getHasQuarterState(){  
        return hasQuarterState;  
    }  
    public String toString(){  
        StringBuffer result = new StringBuffer();  
        result.append("\n주식회사 왕뽑기");  
        result.append("\n2024년형 뽑기 기계");  
        result.append("\n남은 갯수: "+ count + " 개");  
        result.append("\n");  
        result.append("\n현재 상태는 "+ state +"\n");  
        return result.toString();  
    }  
}
```

### 각 상태들 구현 하기

이제 위에서 정의했던, `State` 를  상속받아서 구체적인 상태를 구현하는 구현 클래스를 만듭니다.

각 상태는 어떤 행동 시에 어떤 `State`로 이동할지에 대한 상태 전이를 정의 합니다. 그러기 위해서 생성자에서 모든 상태들을 가지고, `getter`로 상태들을 반환하는 GumballMachine 을 DI 해줍니다.

#### 동전이 있는 상태

🗅 **<span style="color: #c03a92">class HasQuaterState</span>**
```java
package src.state.gumballstate;  
  
public class HasQuaterState implements State {  
  
    private GumballMachine gumballMachine = null;  
  
    public HasQuaterState(GumballMachine _gumballMachine) {  
        this.gumballMachine = _gumballMachine;  
    }  
  
    //동전 넣기  
    @Override  
    public void insertQuater() {  
        System.out.println("동전을 또 넣을 수 없습니다");  
    }  
  
    //동전 반환하기  
    @Override  
    public void ejectQuater() {  
        System.out.println("동전이 반환되었습니다");  
        gumballMachine.setState(gumballMachine.getNoQuarterState());  
    }  
  
    //손잡이 돌리기  
    @Override  
    public void turnCrank() {  
        System.out.println("손잡이를 돌렸습니다...");  
        gumballMachine.setState(gumballMachine.getSoldOutState());  
    }  
  
    //껌볼 내보내기  
    @Override  
    public void dispense() {  
        System.out.println("껌볼이 나오지 않았습니다");  
    }  
  
    public void refill(){  
  
    }  
    public String toString(){  
       return "손잡이를 돌리기 기다리는 중";  
  
    }  
}
```

#### 동전이 없는 상태

🗅 **<span style="color: #c03a92">class NoQuaterState</span>**

```java
package src.state.gumballstate;  
  
public class NoQuaterState implements State{  
  
    private GumballMachine gumballMachine = null;  
  
    public NoQuaterState (GumballMachine gumballMachine){  
        this.gumballMachine = gumballMachine;  
    }  
  
    @Override  
    public void insertQuater() {  
        System.out.println("동전을 넣으셨습니다.");  
        gumballMachine.setState(gumballMachine.getHasQuarterState());  
    }  
  
    @Override  
    public void ejectQuater() {  
        System.out.println("동전을 넣지 않았습니다");  
    }  
  
    @Override  
    public void turnCrank() {  
        System.out.println("손잡이를 돌렸지만, 동전이 없습니다");  
    }  
  
    @Override  
    public void dispense() {  
        System.out.println("먼저 돈을 넣어야 합니다");  
    }  
  
    public void refill(){  
  
    }  
  
    public String toString(){  
        return "동전을 기다리는 중";  
    }  
  
}
```


#### 껌볼을 꺼낸 상태

🗅 **<span style="color: #c03a92">class SoldState</span>**

```java
package src.state.gumballstate;  
  
public class SoldState implements State{  
  
    private GumballMachine gumballMachine = null;  
  
    public SoldState(GumballMachine gumballMachine){  
        this.gumballMachine = gumballMachine;  
    }  
  
    @Override  
    public void insertQuater() {  
        System.out.println("잠깐만 기다려 주세요. 알맹이가 나가고 있습니다.");  
    }  
  
    @Override  
    public void ejectQuater() {  
        System.out.println("이미 알맹이를 뽑았습니다.");  
    }  
  
    @Override  
    public void turnCrank() {  
        System.out.println("손잡이는 한번만 돌려주세요.");  
    }  
  
    @Override  
    public void dispense() {  
        gumballMachine.releaseBall();  
        if(gumballMachine.getCount()>0){  
            gumballMachine.setState(gumballMachine.getNoQuarterState());  
        }  
        else{  
            System.out.println("이런, 껌볼이 다 떨어졌습니다!");  
        }  
    }  
  
    @Override  
    public void refill() {  
  
    }  
  
    public String toString(){  
        return "껌볼을 내보내는 중";  
    }  
}
```

#### 껌볼이 매진된 상태

🗅 **<span style="color: #c03a92">class SoldOutState</span>**
```java
package src.state.gumballstate;  
  
public class SoldOutState implements State{  
  
    private GumballMachine gumballMachine = null;  
    public SoldOutState(GumballMachine _gumballMachine){  
        this.gumballMachine = _gumballMachine;  
    }  
  
    @Override  
    public void insertQuater() {  
        System.out.println("껌볼이 다 떨어져서 동전을 넣을 수 없습니다");  
    }  
  
    @Override  
    public void ejectQuater() {  
        System.out.println("동전을 넣지 않았기 때문에 반환할 수 없습니다");  
    }  
  
    @Override  
    public void turnCrank() {  
        System.out.println("손잡이를 돌렸지만, 껌볼이 없습니다");  
    }  
  
    @Override  
    public void dispense() {  
        System.out.println("껌볼이 나오지 않았습니다");  
    }  
  
    @Override  
    public void refill() {  
        gumballMachine.setState(gumballMachine.getNoQuarterState());  
    }  
  
    public String toString(){  
        return "껌볼이 다 떨어졌습니다.";  
    }  
}
```

### Main 에서 실행하기

이제 상태머신을 생성하고, 알맹이의 갯수에 대한 정보를 생성자를 통해 설정 합니다. 그러면, `GumballMachine`의 생성자에서 조건문을 통해 지금 현재 상태가 결정이 됩니다.

결정된 현재 상태에 따라 `insertQuater`, `turnCrank` 등의 함수가 호출되며, 로직이 진행이 됩니다. 그러면서 알맹이가 점점 줄어들고, 결국은 매진된 상태로 상태가 전이되며, 껌볼 머신의 작동이 중지가 됩니다.

(`refill()` 이라는 메서드를 통해서 껌볼을 더 넣어주지만 넣지 않으면 매진 상태로 종료되게 됩니다)

🗅 **<span style="color: #c03a92">class Main</span>**

```java
package src.state.gumballstate;  
  
public class Main {  
    public static void main(String[] args) {  
        GumballMachine gumballMachine = new GumballMachine(2);  
  
        System.out.println(gumballMachine);  
  
        gumballMachine.insertQuater();  
        gumballMachine.turnCrank();  
  
        System.out.println(gumballMachine);  
  
        gumballMachine.insertQuater();  
        gumballMachine.turnCrank();  
        gumballMachine.insertQuater();  
        gumballMachine.turnCrank();  
  
        gumballMachine.refill(5);  
        gumballMachine.insertQuater();  
        gumballMachine.turnCrank();  
  
        System.out.println(gumballMachine);  
    }  
}
```



## 정리

저는 현재 주로 `Unity`를 통해서 게임을 개발하고 있습니다. 게임을 개발할 때, 상태 패턴은 많이 쓰이는 패턴 중 하나 입니다. 복잡한 플레이어 컨트롤 또는 보스AI 를 구현할 때 자주 사용합니다. 

또한 `Unity`의 `Animator Controller` 또한 상태머신으로 구현되어 있습니다. 상태를 정의하고, 어떤 조건에서 다른 상태로 `전이` 될지만 정해주면 알아서 돌아가도록 만들어져 있습니다.

![](/images/Pasted%20image%2020240929003608.png)

상태패턴을 사용하여 상태를 구현하게 되면, `개방폐쇄원칙 (OCP)`를 지키게 됩니다. 특별한 경우가 아니고선, 새로운 기능이 추가될 때, 기존의 `State`들은 수정하지 않으면서, `State`를 상속받는 새로운 `State`만 구현하고, `StateMachine`에 추가만 해주면 됩니다.

처음 유니티로 AI 를 구현할 때, if 문 떡칠로 구현했던 적이 있습니다. 간단한 로직이었는데도, 구현하기가 상당히 까다로웠습니다. 상태패턴을 공부하고 적용한 후로 구조는 좀더 복잡해 졌지만, 한번 만들어놓으면 상대적으로 복잡한 상태들을 최소한의 오류로 개발할 수 있게 되었습니다.