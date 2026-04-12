---
title: "[Unity] Unity PlayerPrefs 관리법"

categories:
  -  Unity
  
tags:
  - [Unity, PlayerPrefs]

toc: true
toc_sticky: true

published: true

date: 2024-07-22
last_modified_at: 2024-07-22
---

## PlayerPrefs 란?

프로그램의 개발에 있어서 데이터는 가장 중요한 요소 입니다. 보통 온라인 게임을 한다고 생각하면, 서버의 DB를 사용하여 데이터를 저장하게 됩니다. 하지만 DB는 커넥션이 많아질수록 불안정하고, 비용도 많이 들어갑니다. 

따라서 중요한 데이터는 서버에 저장을 하게 되고, 그렇게 중요하지 않은 정보들은 Local에 저장을 하게 됩니다.

보통 아이템정보, 플레이어 정보 등이 서버에 저장되고 옵션값, 전투중 유무, 오늘하루 보지 않기 등의 데이터를 Local로 저장하게 됩니다.

`PlayerPrefs`는 Unity에서 제공하는 기능으로 `Key`, `Value`로 데이터를 저장하게 할 수 있는 클래스 입니다.
`PlayerPrefs`는 `int`, `float`, `string`타입의 데이터를 저장하게 됩니다.

## PlayerPrefs 를 사용하며 관리하기

PlayerPrefs를 사용하는 방법은 다음과 같습니다.
```csharp

public class SomeClass
{
	private string someKey = "someKey";

	private void SomeMethod()
	{
		PlayerPrefs.SetInt(someKey, 0);
		PlayerPrefs.GetInt(someKey);
	}
}
```

`key`를 통해 `SetInt()`로 값을 저장하고, `GetInt()`로 값을 가져옵니다.

하지만, 여기에는 문제가 있습니다. 사용하고 싶은 곳에서 Key로 관리하게 되면, Key가 스크립트 중에 막 널부러져서 나중에는 관리를 할 수 없는 상황이 오게 됩니다. 그렇기 때문에 이것을 관리하는 `PlayerPreferenceManager`를 하나 만들어서 관리하고있습니다.

## PlayerPreferenceManager

해당 메니저 클래스는 별 특별한 기능은 없고, Key들을 관리하고, `SetInt()`와 `GetInt()`를 캡슐화하여 참조하도록 하는 클래스 입니다. 유일성을 보장하기 위해 싱글톤 패턴을 사용하여 구현하였습니다.

```csharp
namespace Common.PlayerPreferences
{
    public class PlayerPreferencesManager
    {
        private static PlayerPreferencesManager instance = null;
        public static PlayerPreferencesManager Instance
        {
            get
            {
                if (instance == null)
                {
                    instance = new PlayerPreferencesManager();
                }
                return instance;
            }
        }
        private PlayerPreferencesManager() { }
        
        private CommonMethod method = new CommonMethod();
		        
        #region Option
        private readonly string bgmMuteKey = "BgmMute";
        private readonly string sfxMuteKey = "SfxMute";

        private readonly string bgmValueKey = "BgmValue";
        private readonly string sfxValueKey = "SfxValue";
        #endregion

	    #region Option
		  
	  public void SetBgmMuteStatus(bool _status)
	  {
	      PlayerPrefs.SetInt(bgmMuteKey, method.ConvertBoolToInt(_status));
	  }
	  public void SetSfxMuteStatus(bool _status)
	  {
	      PlayerPrefs.SetInt(sfxMuteKey, method.ConvertBoolToInt(_status));
	  }
	  public bool HasBgmMute()
	  {
	      return method.ConvertIntToBool(PlayerPrefs.GetInt(bgmMuteKey));
	  }
	  public bool HasSfxMute()
	  {
	      return method.ConvertIntToBool(PlayerPrefs.GetInt(sfxMuteKey));
	  }
	  #endregion
	}
}
```

위에 작성한 것 처럼, Key 들을 모두 모아두고, Get() Set() 메서드를 캡슐화하여 관리하도록 만들었습니다.

사용할때는 다음과 같이 사용하면 됩니다.

```csharp
    private void OnBgmSliderValueChanged(float _value)
    {
        // 최소 양수 값으로 설정하여 Log10 함수가 정의되도록 함
        if (_value <= 0)
        {
            _value = 0.0001f;
            buttonBgmMute.SetToggleState(true);
        }
        else
        {
            buttonBgmMute.SetToggleState(false);
        }
        PlayerPreferencesManager.Instance.SetBgmMuteStatus(buttonBgmMute.IsPressed);
        PlayerPreferencesManager.Instance.SetBgmValue(_value);

        mixer.SetFloat("BGM", Mathf.Log10(_value) * 20);
    }
```
