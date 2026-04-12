---
title: "[Clean Architecture] Clean Architecture ISP(Interface Segregation Principle 인터페이스 분리의 원칙)"

categories:
  -  Clean Architecture
  
tags:
  - [Clean Architecture]

toc: true
toc_sticky: true

published: true

date: 2025-03-02
last_modified_at: 2025-03-02
---

## ISP 원칙

ISP 원칙이란 범용적인 인터페이스 보다는 클라이언트가 실제로 사용하는 Interface를 만들어야 한다는 의미로, 인터페이스를 사용에 맞게끔 각기 분리해야 한다는 원칙입니다. 이건 너무 당연한거 아닌가? 라고 할 수 있지만, 실제로 코딩을 하다보면 처음의 설계와는 달라지는 경우가 많기 때문에 어제까지는 ISP를 지키고 있다가도 요구사항이 변경되면서 ISP를 위배하는 경우가 생기기 때문이라고 생각합니다. 

## ISP 원칙 위반 사례

그렇다면 해당 원칙을 위배되는 사례에는 어떤게 있을까요? 해당 예제는 `탈것`이라는 주제로 구성하였습니다.

어제까지는 Vehicle 에 자동차와 자전거만 있었다고 가정합니다. 그래서 `IVehicle`이라는 인터페이스를 만들어서 사용했습니다. 위 코드처럼 앞, 뒤, 좌, 우 로 움직이도록 인터페이스를 설계했다고 가정합니다.

```cs
public interface IVehicle
{
    public void MoveForward();
    public void MoveBackward();
    public void MoveLeft();
    public void MoveRight();
}
```

```cs
namespace ISP
{
    public class Car : IVehicle
    {
        public void MoveBackward()
        {
            //..
        }

        public void MoveForward()
        {
            //..
        }

        public void MoveLeft()
        {
            //..
        }

        public void MoveRight()
        {
            //..
        }
    }
}
```

```cs
namespace ISP
{
    public class Bike : IVehicle
    {
        public void MoveBackward()
        {
            //..
        }

        public void MoveForward()
        {
            //..
        }

        public void MoveLeft()
        {
            //..
        }

        public void MoveRight()
        {
            //..
        }
    }
}
```

 하지만 여기서 기획자가 오더니 오늘부터 탈것에 `기차`가 추가되었다고 통보하였습니다. 그래서 미리 만들어두었던 IVehicle 을 상속시켜 Train을 만들었습니다.

```cs
namespace ISP
{
    public class Train : IVehicle
    {
        public void MoveBackward()
        {
            //..
        }

        public void MoveForward()
        {
            //..
        }

        public void MoveLeft()
        {

        }

        public void MoveRight()
        {

        }
    }
}
```

여기서 잠깐, 상속을 받고 생각해보니 Train은 앞,뒤 로만 움직이고 좌,우 로는 움직일 수 없다는걸 꺠달았습니다. 하지만, 인터페이스를 분리하기 귀찮으니 그냥 사용을 합니다. 

지금의 상황이 ISP를 위반한 상황입니다. Train은 MoveLeft와 MoveRight가 구현이 되어있지 않더라도 인터페이스를 통해 호출을 하기 때문에, 클라이언트에서 문제가 발생할 확률이 엄청 높아지게 됩니다. 

따라서, `IVehicle`인터페이스를 분리하여 `IMoveFoward`, `IMoveLeft`, `IMoveRight`, `IMoveBackward` 처럼 분리하여 구현해야 합니다.

## 결론

인터페이스 분리의 원칙은 개념은 쉽지만, 제대로 지키기에는 어려운것 같습니다. 왜냐하면 요구사항이 바뀔것을 예측할 수 없을 뿐더러, 그것을 예측한다고 하더라도 모든 상황에 대비하여 인터페이스를 분리하는것은 말이 안되기 때문입니다. 즉, 이것은 개발자의 역량과 경험에 달려있다고 봐도 무방할 것 같습니다.

하지만, 지키기 어렵다고 해서 원칙을 배제한다는것은 말이되지 않기 때문에 항상 생각하며 인터페이스를 다뤄야 합니다.