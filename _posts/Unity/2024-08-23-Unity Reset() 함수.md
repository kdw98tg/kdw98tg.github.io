---
title: "[Unity] Reset() 사용법 및 컴포넌트 초기화 방법"

categories:
  -  Unity
  
tags:
  - [Unity, Init, Reset]

toc: true
toc_sticky: true

published: true

date: 2024-08-23
last_modified_at: 2024-08-23
---

## Unity LifeCycle

유니티에서 기본적으로 GameObject에 들어가는 MonoBehaviour 에는 생명주기가 있습니다.

생명주기는 다음과 같습니다.

![Unity LifeCycle](/images/Pasted%20image%2020240823155242.png)

여기서 항상 `Reset()` 함수는 어디에 사용하는거지? 라는 의문이 있었습니다.

## Reset() 함수 사용법

프로젝트를 하다가 어떤 클래스에 `RequrieComponent(typeof(BoxCollider2D))`를 할 일이 있었습니다. 근데 제가 원하는 BoxCollider2D 에는 isTrigger 체크가 항상 되어 있어야 했습니다. 

물론 Awake() 에서 변수 할당 후 Start() 에서 실행하게 하면 되지만, 제가 원한것은 RequireComponent 가 실행될 때 자동으로 제가 세팅한 값으로 초기화가 되는 것 이었습니다.

찾던도중 Reset() 함수로 하는 방법을 알아냈습니다.

🗅 **<span style="color: #c03a92">class SomeClass</span>**
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(BoxCollider2D))]

public class SomeClass : MonoBehaviour
{
	private BoxCollider2D boxCollider = null;
		
	private void Reset()
	{
		boxCollider = GetComponent<BoxCollider2D>();
		boxCollider.isTrigger= true;
	}
}
```

해당 클래스는 이 클래스가 게임오브젝트에 붙을 때 BoxCollider2D 를자동으로 붙여줍니다. 또한 스크립트가 붙으면 바로 `Reset()`함수가 실행이 되며, 제가 설정한 값대로 컴포넌트에 반영이 됩니다.

## 인스펙터에서의 Reset()

이렇게 Reset() 함수를 쓰고 카톡방에 공유하니, 이미 알고있는 분들이 많았습니다. 그리고 인스펙터에서 컴포넌트 오른쪽 상단에 점 세개짜리 메뉴를 누르면 나오는 Reset 이 해당 Reset() 함수를 돌리는 것 이었다는 것도 알게 되었습니다.

![](/images/Pasted%20image%2020240823160332.png)

따라서 해당 Reset 을 누르면 제가 새로 정의한 Reset() 함수가 돌아가면서 초기화가 진행되게 됩니다.

이 기능을 몰랐을 때는 언제 하나씩 다 설정해주나 했는데, 해당 기능을 알고나니 손쉽게 초기화를 해줄 수 있겠네요.

역시 아는 만큼 보인다 라는 말이 맞는 거 같습니다.