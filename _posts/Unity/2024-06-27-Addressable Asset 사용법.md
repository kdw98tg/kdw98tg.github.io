---
title: "[Unity] Addressable Asset 활용법"

categories:
  -  Unity
  
tags:
  - [Unity, Addressable]

toc: true
toc_sticky: true

published: true

date: 2024-06-27
last_modified_at: 2024-06-27
---
## 에셋 관리 방법

에셋을 관리하는 방법에는 여러가지가 있습니다.

제일 고전적인 에셋 관리 방법은 Unity의 `ResourceManager`를 통해 관리하는 방법입니다. 유니티의 예약 폴더에 `Resource`라는 폴더를 만들어 두고, 해당 폴더 아래에 에셋들을 보관하며, 에셋을 불러오고 싶을때는 `Resources.Load<>`함수를 통해 불러오게 됩니다.

하지만 이 방법은 로컬폴더에만 접근가능하다는 단점이 있어, 어떤 에셋을 업데이트할 때 마다 버전 업데이트를 해줘야 한다는 치명적인 단점이 있습니다.

이것을 보완하고자 나온 방법이 `Asset Bundle` 이라는 시스템 입니다. 이것은 에셋들을 서버에서 관리하고, 에셋 업데이트 시 게임 자체의 버전 업데이트가 필요 없도록 하고, 서버에서 에셋을 내려받는 시스템 입니다. 하지만 `Asset Bunlde`은 메모리 관리가 까다롭고, 런타임 성능 문제가 있었습니다.

또 이것을 보완하기 위해 나온 것이 `Addressable Asset`입니다. `Asset Bundle`의 메모리 문제를 해결하고, 유연한 로딩 및 관리로 더욱 유연하게 에셋을 사용할 수 있는 시스템 입니다.

## Addressable Asset 사용하는 방법

우선, 이것을 사용하기 위해서는 **Window** > **PackageManager** > **Unity Registry** 로 이동하셔서 **Addressable Asset** 을 다운받아야 합니다.

Addressable Asset을 다운받고 나면, Addressable 에 접근할 수 있는 메뉴가 생성됩니다.

![Addressable Import](/images/Pasted%20image%2020240702124051.png)

이렇게 하면 Addressable 을 사용할 준비가 완료되었습니다.

**WIndow** > **Asset Management** > **Addressables** > **Group** 을 통해 새로운 Addressable Gorup 을 지정해 줍니다.

![Addressable-Group](/images/Pasted%20image%2020240702124218.png)

이렇게 하면, Addressable 을 사용할 준비가 완료되었습니다.

## Addressable 등록하기

이제, Addressable Asset을 사용하기 위해서 Addressable 로 관리할 오브젝트들을 설정해 봅시다.

Addressable 을 사용하기 위해서는 GameObject를 Prefab 으로 만들어주고, 해당 Prefab을 Addressable Group으로 드래그 앤 드롭 해주면 됩니다.

저는 `UI`라는 새로운 Addressable Group을 만들어서 UI 들을 등록 해놨습니다.
![](Pasted%20image%2020240702124533.png)

오른쪽에보면 `preload`라고 적혀있는 곳이 있는데, 여기는 `Label`을 적는 칸이다.

여기서 지정한 Label 을 통해 코드에서 로드 타이밍을 정할 수 있다.
(Label은 다중으로 설정 가능함)

## Addressable 가져오기

이제 코드로 Addressable Asset을 가져오도록 합니다.
Addressable Asset 을 가져올떄는 항상 비동기로 가져오도록 만들어져 있습니다.

따라서 callback 을 통해서 완료되었다는 사실을 전달해줄 수 있도록 합니다.

### 하나만 지정해서 가져오기

```cs
private void LoadAsync<T>(string _key, Action<T> _callback = null) where T : UnityEngine.Object

{
	string loadKey = _key;

	if (_key.Contains(".sprite"))
	{
		loadKey = $"{_key}[{_key.Replace(".sprite", "")}]";
	}
	
	var asyncOperation = Addressables.LoadAssetAsync<T>(loadKey);
	asyncOperation.Completed += (op) =>
	{
		//캐시 확인
	
		if (resources.TryGetValue(_key, out UnityEngine.Object resource))
		{
			_callback?.Invoke(op.Result);
			return;
		}
	
	resources.Add(_key, op.Result);
	_callback?.Invoke(op.Result);
	
	};	
}

```

### 전체 로드
```csharp
    public void LoadAllAsync<T>(string label, Action<string, int, int> callback) where T : UnityEngine.Object
    {
        var opHandle = Addressables.LoadResourceLocationsAsync(label, typeof(T));
        opHandle.Completed += (op) =>
        {
            int loadCount = 0;

            int totalCount = op.Result.Count;

            foreach (var result in op.Result)
            {
                if (result.PrimaryKey.Contains(".sprite"))
                {
                    LoadAsync<Sprite>(result.PrimaryKey, (obj) =>
                    {
                        loadCount++;
                        Debug.Log(result.PrimaryKey + " " + loadCount + " / " + totalCount);
                        callback?.Invoke(result.PrimaryKey, loadCount, totalCount);
                    });
                }
                else
                {
                    Debug.Log(result.PrimaryKey);
                    LoadAsync<T>(result.PrimaryKey, (obj) =>
                    {
                        loadCount++;
                        Debug.Log(result.PrimaryKey + " " + loadCount + " / " + totalCount);
                        callback?.Invoke(result.PrimaryKey, loadCount, totalCount);
                    });
                }
            }
        };
    }
}
```

이렇게하면, 아래와 같이 사용할 수 있습니다.

```csharp
ResourceManager.Instance.LoadAllAsync<Object>("preload", (primaryKey, loadCount, totalCount) =>
{
	//프로그레스바 매핑하기
	if (loadCount == totalCount)
	{
		Debug.Log("Loaded");
	}
});
```

## 개선사항
현재는 로컬에 저장되어 있지만, 이 Addressable 파일들을 서버로 빼내서, 앱스토어에 업데이트 하지 않아도, Resource 들을 바꿀 수 있도록 해야 합니다.