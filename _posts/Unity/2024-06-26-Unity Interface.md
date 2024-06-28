---
title: "[Unity] Interface를 활용하여 느슨한 결합 만들기"

categories:
  -  Categories
  
tags:
  - [Tags1, Tags2]

toc: true
toc_sticky: true

published: true

date: 2024-06-26
last_modified_at: 2024-06-26
---

interface, inherit  등등 객체지향에서 등장하는 개념들은 처음에 이해하기는 어렵지만, '객체지향의 꽃' 이라고 할 만큼 중요한 개념입니다.

절차지향에서는 없는 바로 '다형성'이라는 개념 덕분입니다.

이 다형성이라는 녀석의 정의는 다음과 같습니다.

> 다형성이란, 이름에서 유추할 수 있듯이, 어떤 객체의 속성이나 기능이 상황에 따라 여러가지 형태를 지닐 수 있는 성질을 말합니다.

![](/images/Pasted%20image%2020240626173431.png)

위 사진과 같이 `로미오 역할` 에 `장동건`, `원빈` 이 들어갈 수 있는것이 예 입니다.

즉 하나의 틀을 정해놓고(interface 정의) 해당 틀을 상속받는 클래스(구현 클래스)는 해당 틀을 구현하기만 하면, 같은 종류의 class 가 된다는 뜻입니다.

말이 어렵네요. 예제를 하나 가져왔습니다.

예제는 Interface 하면 가장 많이 사용되는 `Item` 으로 가져왔습니다.

우선, Item 은 `Use()` 라는 함수를 공통으로 사용한다고 가정하겠습니다.
그것을 바탕으로 인터페이스`IItem`을 만듭니다. (보통 Interface를 만들면 이름 앞에 'I'를 붙이는 것이 보통입니다. Java 진영에서는 이름 뒤에 Impl 을 붙이는 것으로 알고 있습니다.)

🗅 **<span style="color: #c03a92">Interface IItem</span>**

```csharp
public interface IItem
{
	public void Use();
}
```
Item 들이 공통적으로 사용할 `Use()` 함수를 만듭니다. Interface 에서는 구현이 없는 함수의 틀만 만들어 놓습니다.

그 뒤, IItem 을 상속받아 Use 함수를 구현할 아이템들을 만들겠습니다.

Item의 종류는 `HealItem`, `SpeedItem`, `BombItem` 이렇게 3가지만 만들겠습니다.

🗅 **<span style="color: #c03a92">class HealItem</span>**

```csharp
public class HealItem : MonoBehaviour, IItem
{
	public void Use()
	{
		Debug.Log("HealItem 사용");
	}
}
```

🗅 **<span style="color: #c03a92">class SpeedItem</span>**

```csharp
public class SpeedItem : MonoBehaviour, IItem
{
	public void Use()
	{
		Debug.Log("SpeedItem 사용");
	}
}
```

🗅 **<span style="color: #c03a92">class BombItem</span>**

```csharp
public class BombItem : MonoBehaviour, IItem
{
	public void Use()
	{
		Debug.Log("BombItem 사용");
	}
}
```


그리고 나서, Player를 생성해서 해당 아이템과 충돌 시 , 해당 아이템을 사용할 수 있도록 코드를 짭니다.
그리고 Item 들은 Trigger로 구현되어 있어서, OnTriggerEnter 에서 사용합니다.

🗅 **<span style="color: #c03a92">class Player</span>**

```csharp
public class Player : MonoBehaviour
{
	public void OnTriggerEnter(Collider _other)
	{
		IItem item = GetComponent<IItem>();
		item?.Use();
	}
}
```

Player 는 위와같이 구현하면 됩니다.

이렇게 구현하고, Unity에서 실행시키면 `IItem` 을 상속받는 아이템이라면 Use() 할 수 있게 됩니다.

언틋보면, interface를 사용하면서 더 복잡해진 것 같지만, 엄청난 확장성이 있는 코드를 만들게 된 것 입니다.

`if`문으로 분기를 나누어서 Item.Use()를 구현하였다면, Item의 갯수만큼 `if`문이 생기게 되지만, interface 를 활용하여 만들면, OnTirggerEnter 즉, 실행부의 수정 없이, 새로운 Item을 만들고, IItem을 상속받은 뒤에 Use() 함수를 만들어주기만 하면 됩니다.

