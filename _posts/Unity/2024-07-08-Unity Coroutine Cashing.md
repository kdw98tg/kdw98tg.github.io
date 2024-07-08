---
title: "[Unity] 코루틴 관리를 도와주는 CoroutineCashManager"

categories:
  -  Unity
  
tags:
  - [Unity, Coroutine, cash]

toc: true
toc_sticky: true

published: true

date: 2024-07-08
last_modified_at: 2024-07-08
---

Unity 에서 작업을 하시다 보면 `Coroutine`이라는 녀석을 많이 사용합니다.

Coroutine 에 관한 내용은 아래를 참조 하시면 됩니다.

[👉👉👉  Unity Coroutine 게시글](https://kdw98tg.github.io/unity/Coroutine-%ED%99%9C%EC%9A%A9%EB%B2%95/)


(본 게시글은 구름고래님의 블로그를 참조하였습니다.)

[👉👉👉  구름고래의 이스터에그](http://blog.naver.com/kch8246)
## Corotuine의 문제점
현재의 Unity는 C# 의 8.0 버전을 사용하고 있습니다. C#의 8.0 버전에는 Coroutined을 사용하려면 필수 키워드인 `yield` 에 문제가 있다고 합니다.

여기서 예상치 못한 메모리 누수가 생겨서 반복문을 사용하거나, 코루틴을 많이 사용해야 할 때에는 문제가 발생할 수 있다고 합니다.(현재 최신 C# 버전에는 수정되어 있습니다.)

그 외에도 Coroutine 을 사용할 때 불필요하게 가비지를 생성하게 되는 문제가 있습니다.
```csharp
private IEnumerator SomeCoroutine()
{
	while(true)
	{
		yield return new WaitForSeconds(1f);
	}
}
```
위와 같은 코드 입니다. 위 코드는 1초마다 `WaitForSeconds`라는 객체를 계속 생성하여 가비지를 발생시킵니다.

이를 해결하기위해 간단하게 다음과 같이 전역변수로 빼는 방법도 있긴 합니다.
```csharp
private WaitForSeconds waitSecond = new WaitForSeconds(1f);
private IEnumerator SomeCoroutine()
{
	while(true)
	{
		yield return waitSecond;
	}
}
```

하지만 이렇게 하면 코루틴을 사용할 때 마다 불필요하게 전역변수를 만들어줘야 한다는 단점이 있습니다.

## 해결법

이것을 해결하기위해 따로 Script를 하나 만들어 줍니다. 해당 스크립트는 `yield return`과 함께 쓰일 객체들을 미리 만들어 두고, `Dictionary`에 저장해 캐싱을 합니다.

```csharp
private static class CoroutineCashManager
{
	public static readonly Dictionary<float, WaitForSeconds> waitForSeconds = new Dictionary<float, WaitForSeconds>();
	public static WaitForSeconds(float _seconds)
	{
		WaitForSeconds wfs;
		if(!waitForSeconds.TryGetValue(_seconds, out wfs))
		{
			waitForSeconds.Add(_seconds, wfs = new WaitForSeconds(_seconds));
		}
	}
}
```

위와같이 Dictionary를 만들어두고 아래와 같이 사용하면 됩니다.

```csharp
private IEnumerator SomeCoroutine()
{
	while(true)
	{
		yield return CoroutineCashManager.WaitForSeconds(1f);
	}
}
```

이렇게하면 불필요한 메모리 낭비 없이 yield return 다음에 올 객체들을 캐싱해서 사용할 수 있습니다.