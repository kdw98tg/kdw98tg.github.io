---
title: "[Unity] DragHandler를 이용하여 오브젝트 옮기기"

categories:
  -  Unity
  
tags:
  - [Unity, Drag]

toc: true
toc_sticky: true

published: true

date: 2024-08-23
last_modified_at: 2024-08-23
---
## Drag 기능 구현하기

이번 프로젝트를 진행하면서, Tilemap 안의 오브젝트를 움직이는 기능이 있었습니다. 해당 기능은 `아이러브커피`에도 존재하는 기능입니다.
![Drag 기능](/images/Pasted%20image%2020240823134546.png)

`Remove`버튼과 `Confirm`버튼은 단순히 클릭이벤트가 일어날 때 로직이 발동되면 되지만, `Move`의 경우에는 다릅니다.

일단 버튼 이벤트와 다른점은, 버튼 이벤트는 마우스 포인터가 내려갔다가 올라갈 때 발동됩니다. 제가 원하는 Drag 기능과는 다릅니다.

Drag는 마우스 포인터가 내려갔을 때, 발생되고 마우스 포인터가 다시 올라갈 때 특정 이벤트가 발동되어야 합니다.

처음에는 그냥 Button 이벤트들로만 해보려고 했다가 안되서 찾아보니 유니티에서 제공하는 `IDragHandler`라는 이벤트가 존재했습니다. 이것으로 구현한 예제를 가져왔습니다.


## IDragHandler

`IDragHandler`는  Unity에서 만들어놓은 이벤트 시스템 인터페이스 입니다.

이 인터페이스를 드래그 하기를 원하는 객체에 상속시키고 `OnBeginDrag`, `OnDrag`, `OnDragEnd` 메서드를 구현해주면 됩니다. 저는 UI 가 움직여야 하기 떄문에 UI 객체에 해당 인터페이스를 달아줬습니다.

그리고 결합도를 낮추기 위해 callback 으로 연결하여 상위 버튼들을 관리하는 객체에서 관리하도록 하였습니다.

🗅 **<span style="color: #c03a92">class UI_GridController</span>**
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;
using UnityEngine.EventSystems;

public class UI_GridController : UI_Base
{
	private UI_GridController_Button_Move buttonMove = null;
	private UI_GridController_Button_Confirm buttonConfirm = null;
	private UI_GridController_Button_Remove buttonRemove = null;
	
	private Vector3 dragOffset;
	private Building building = null;
	private Action updateTileAreaCallback = null;
	
	protected override void Awake()
	
	{
		base.Awake();
		
		buttonMove = GetComponentInChildren<UI_GridController_Button_Move>();
		buttonConfirm = GetComponentInChildren<UI_GridController_Button_Confirm>();
		buttonRemove = GetComponentInChildren<UI_GridController_Button_Remove>();
	}
	
	//여기서 버튼 이벤트 등록해줘야하는데
	
	//나중에 GridController에서 DI 받아서 해주던가 하자
	
	public void Init(Action _updateTileAreaCallback, UnityAction _confirmAction, UnityAction _removeAction)
	{
		buttonMove.Init(OnBeginDrag, OnDrag, OnDragEnd);
		updateTileAreaCallback = _updateTileAreaCallback;
		buttonConfirm.SetOnClickEvent(_confirmAction);
		buttonRemove.SetOnClickEvent(_removeAction);
	}
	
	#region MovebuttonDrag
	
	private void OnBeginDrag(PointerEventData _eventData)
	{
	//드래그 시작할 때 발생하는 로직 정의
	}
	
	private void OnDrag(PointerEventData _eventData)
	{
	//드래그 중에 계속 발생되는 로직 정의
	}
	
	private void OnDragEnd(PointerEventData _eventData)
	{
	//드래그가 끝났을 때 발생되는 로직 정의
	}
}
#endregion
```

🗅 **<span style="color: #c03a92">class UI_GridController_Button_Move</span>**
```csharp
using UnityEngine.EventSystems;
using UnityEngine;
using System;

public class UI_GridController_Button_Move : UI_Button, IBeginDragHandler, IDragHandler, IEndDragHandler

{
	private Action<PointerEventData> onBeginDragCallback = null;
	private Action<PointerEventData> onDragCallback = null;
	private Action<PointerEventData> onEndDragCallback = null;
	
	
	public void Init(Action<PointerEventData> _onBeginDragCallback, Action<PointerEventData> _onDragCallback, Action<PointerEventData> _onEndDragCallback)
	{
		this.onBeginDragCallback = _onBeginDragCallback;
		
		this.onDragCallback = _onDragCallback;
		
		this.onEndDragCallback = _onEndDragCallback;
	}
	
	  
	
	public void OnBeginDrag(PointerEventData eventData)
	{
		onBeginDragCallback?.Invoke(eventData);
	}
	
	  
	
	public void OnDrag(PointerEventData eventData)
	{
		onDragCallback?.Invoke(eventData);
	}
	
	  
	
	public void OnEndDrag(PointerEventData eventData)
	{
		onEndDragCallback?.Invoke(eventData);
	}
}
```

Drag가 되어야 하는 객체에 `IDragable`인터페이스를 상속시키고, 정의부에서 실행해야 하는 로직을 구현해주면 바로 실행이 됩니다.

원래는 코루틴을 막 섞어서 구현을 했는데, 이게 훨씬 간단하게 구현되는것 같습니다. 또한, Drag 중이 아닐때는 불필요하게 매 프레임 연산을 하지 않아도 되기 때문에 더 좋은 것 같습니다.

![move 구현](/images/Pasted%20image%2020240823140230.png)