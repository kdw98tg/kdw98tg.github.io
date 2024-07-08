---
title: "[Unity] SoundPooling 과 Resource 관리를 통한 SoundManager 만들기"

categories:
  -  Unity
  
tags:
  - [Unity, C#, SoundManager]

toc: true
toc_sticky: true

published: true

date: 2024-04-03
last_modified_at: 2024-04-03
---

이전 `SoundManager` 를 만들었을 때의 문제점을 보완해서 다시 만들게 되었습니다.

👉[마시남의 기술 블로그 - 이전 SoundManager](https://kdw98tg.github.io/unity/Unity-SoundManager/)👈


위 사운드메니저에는 다음과 같은 문제가 있었습니다.

1. 사운드매니저가 리소스를 한번에 가지고 있었음
2. SoundPool 방식이 비효율적이었음(시작할 때 한번에 대량생산, SetActive 값 변경)
3. SoundManager가 굳이 GameObject가 되지 않아됨

이러한 문제점들을 개선하여 재구성하게 되었습니다.

구조는 아래와 같습니다.

![SoundManager 구조도](/images/Pasted%20image%2020240403141632.png)

`GameManager`는 `StageManager` 에게 씬을 변경하라고 지시하고, 그때에 SoundManager를 사용하여 음악을 재생하는 방식입니다.

## 구현

### Enums 만들기

🗅 **<span style="color: #c03a92">Enums</span>**
```cs
namespace Enums
{
    public enum STAGES
    {
        STAGE01,
        STAGE02,
        STAGE03,
        STAGE04,
    }

    public enum PREFABS
    {
        AUDIO_PREFAB,
    }

    public enum BGMS
    {
        //Stage01
        STAGE_01_START,

        //Stage02
        STAGE_02_START,

        //Stage03
        STAGE_03_START,
        STAGE_03_ENTER,

        //Stage04
        STAGE_04_BATTLE_PAGE_01,
        STAGE_04_BATTLE_PAGE_02,
        STAGE_04_BATTLE_PAGE_03,
        STAGE_04_BATTLE_PAGE_04,
    }

    public enum SFXS 
    {
        GUN,
        SWORD,
        ARROW
    }
}

```

사용할 리소스에 대한 Enum 들을 생성합니다. 해당 Enum 들을 `ResourceManager`에서 하드코딩으로 매핑할 예정입니다.

### Stage 변경 로직 만들기

우선, Sound를 구현하기 전에 Stage 변경 로직을 구현합니다.

🗅 **<span style="color: #c03a92">StageManager</span>**
```cs
using UnityEngine;
using UnityEngine.SceneManagement;
using Enums;

public class StageManager : MonoBehaviour
{
    public void SetStage(STAGES _stage)
    {
        SceneManager.LoadScene((int)_stage);
    }
}

```

`StageManager` 는 `SetStage` Enum 을 받아서, buildIndex에 따라 매핑되어 씬을 전환 시킵니다.


🗅 **<span style="color: #c03a92">class GameManager</span>**

```cs
using UnityEngine;
using Enums;


public class GameManager : MonoBehaviour
{
    private StageManager stageManager = null;

    private void Awake()
    {
        stageManager = GetComponentInChildren<StageManager>();
    }

    private void Update()
    {
        //BGMs
        if (Input.GetKeyDown(KeyCode.Q))
        {
            stageManager.SetStage(STAGES.STAGE01);
        }
        else if (Input.GetKeyDown(KeyCode.W))
        {
            stageManager.SetStage(STAGES.STAGE02);
        }
        else if (Input.GetKeyDown(KeyCode.E))
        {
            stageManager.SetStage(STAGES.STAGE03);
        }
        else if (Input.GetKeyDown(KeyCode.R))
        {
            stageManager.SetStage(STAGES.STAGE04);
        }
    }
}
```

`GameManager`에서는 Q,W,E,R, 을눌러 총 4가지의 스테이지를 변경하도록 합니다.

### ResourceManager 구현

이제 스테이지 변경 로직은 완료하였으니, `ResourceManager`를 구현하여 필요한 리소스들을 불러올 수 있도록 만듭니다.

Dictionary를 사용하고, Load 가 필요한 리소스의 이름을 Key 값으로 저장합니다.

또한, 여기서 Enum과 리소스의 이름을 매핑하는 작업을 합니다. 

당연하지만, Load 할 Resource 들은 `Resources`폴더 안의 경로에 있어야 합니다.,

🗅 **<span style="color: #c03a92">class ResourceManager</span>**
```cs
using System.Collections.Generic;
using UnityEngine;
using Enums;

public class ResourceManager
{
    private static ResourceManager instance = null;
    public static ResourceManager Instance
    {
        get
        {
            if (instance == null)
            {
                instance = new ResourceManager();
            }
            return instance;
        }
    }
    private ResourceManager() { }

    private Dictionary<string, GameObject> prefabs = new Dictionary<string, GameObject>();
    private Dictionary<string, AudioClip> audioClips = new Dictionary<string, AudioClip>();

    //Paths
    private readonly string PATH_BGM_STAGE_01 = "Audios\\BGM\\Stage01\\";
    private readonly string PATH_BGM_STAGE_02 = "Audios\\BGM\\Stage02\\";
    private readonly string PATH_BGM_STAGE_03 = "Audios\\BGM\\Stage03\\";
    private readonly string PATH_BGM_STAGE_04 = "Audios\\BGM\\Stage04\\";

    private readonly string PATH_SFX_WEAPON = "Audios\\SFX\\Weapon\\";

    private readonly string PATH_PREFAB = "Prefabs\\";

    public void ClearAudioResources()
    {
        audioClips.Clear();
    }

    public GameObject GetPrefab(PREFABS _prefabs)
    {
        GameObject soundPrefab = null;
        switch (_prefabs)
        {
            case PREFABS.AUDIO_PREFAB:
                soundPrefab = LoadPrefab(_prefabs, PATH_PREFAB);
                break;

            default:
                Debug.LogError(this + "UnExpectedValue");
                break;
        }
        return soundPrefab;
    }

    public GameObject LoadPrefab(PREFABS _prefabs,string _path)
    {
        GameObject result = null;
        string prefabKey = ConvertEnumToPrefabName(_prefabs);
        if (prefabs.ContainsKey(prefabKey))
        {
            result = prefabs[prefabKey];
        }
        else
        {
            GameObject loadPrefab = Resources.Load<GameObject>(_path + prefabKey);
            if (loadPrefab != null)
            {
                prefabs.Add(prefabKey, loadPrefab);
                result = prefabs[prefabKey] = loadPrefab;
            }
        }
        return result;
    }
    public AudioClip GetAudioClip(BGMS _bgm)
    {
        AudioClip result = null;
        switch (_bgm)
        {
            //stage01
            case BGMS.STAGE_01_START:
                result = LoadAudioClip(_bgm, PATH_BGM_STAGE_01);
                break;

            //stage02
            case BGMS.STAGE_02_START:
                result = LoadAudioClip(_bgm, PATH_BGM_STAGE_02);
                break;

            //stage03
            case BGMS.STAGE_03_START:
            case BGMS.STAGE_03_ENTER:
                result = LoadAudioClip(_bgm, PATH_BGM_STAGE_03);
                break;

            //stage04
            case BGMS.STAGE_04_BATTLE_PAGE_01:
            case BGMS.STAGE_04_BATTLE_PAGE_02:
            case BGMS.STAGE_04_BATTLE_PAGE_03:
            case BGMS.STAGE_04_BATTLE_PAGE_04:
                result = LoadAudioClip(_bgm, PATH_BGM_STAGE_04);
                break;

            default:
                Debug.LogError("UnExpectedValue");
                break;
        }
        return result;
    }

    public AudioClip GetAudioClip(SFXS _sfx)
    {
        AudioClip result = null;
        switch (_sfx)
        {
            case SFXS.GUN:
                result = LoadAudioClip(_sfx,PATH_SFX_WEAPON);
                break;

            case SFXS.SWORD:
                result = LoadAudioClip(_sfx, PATH_SFX_WEAPON);
                break;

            case SFXS.ARROW:
                result = LoadAudioClip(_sfx, PATH_SFX_WEAPON);
                break;

            default:
                Debug.LogError("UnExpectedValue");
                break;
        }
        return result;
    }

    public AudioClip LoadAudioClip(BGMS _bgm, string _path)
    {
        AudioClip result = null;
        string audioKey = ConvertEnumToAudioClipName(_bgm);
        if (audioClips.ContainsKey(audioKey))
        {
            result = audioClips[audioKey];
        }
        else
        {
            AudioClip loadAudioClip = Resources.Load<AudioClip>(_path + audioKey);
            if (loadAudioClip != null)
            {
                audioClips.Add(audioKey, loadAudioClip);
                result = audioClips[audioKey] = loadAudioClip;
            }
        }
        return result;
    }

    public AudioClip LoadAudioClip(SFXS _bgm, string _path)
    {
        AudioClip result = null;
        string audioKey = ConvertEnumToAudioClipName(_bgm);
        if (audioClips.ContainsKey(audioKey))
        {
            result = audioClips[audioKey];
        }
        else
        {
            AudioClip loadAudioClip = Resources.Load<AudioClip>(_path + audioKey);
            if (loadAudioClip != null)
            {
                audioClips.Add(audioKey, loadAudioClip);
                result = audioClips[audioKey] = loadAudioClip;
            }
        }
        return result;
    }

    #region ConvertEnumsToString
    public string ConvertEnumToPrefabName(PREFABS _prefabs)
    {
        string result = string.Empty;
        switch (_prefabs)
        {
            case PREFABS.AUDIO_PREFAB:
                result = "P_AudioPlayer";
                break;

            default:
                Debug.LogError(this + "UnExpectedValue");
                break;

        }
        return result;
    }
    public string ConvertEnumToAudioClipName(BGMS _bgm)
    {
        string result = string.Empty;
        switch (_bgm)
        {
            case BGMS.STAGE_01_START:
                result = "BGM_Stage01";
                break;

            case BGMS.STAGE_02_START:
                result = "BGM_Stage02";
                break;

            case BGMS.STAGE_03_START:
                result = "BGM_Stage03";
                break;

            case BGMS.STAGE_03_ENTER:
                result = "BGM_Stage03_Enter";
                break;

            case BGMS.STAGE_04_BATTLE_PAGE_01:
                result = "BattleBGM1";
                break;

            case BGMS.STAGE_04_BATTLE_PAGE_02:
                result = "BattleBGM2";
                break;

            case BGMS.STAGE_04_BATTLE_PAGE_03:
                result = "BGM_Stage04_Boss_Page01";
                break;

            case BGMS.STAGE_04_BATTLE_PAGE_04:
                result = "BGM_Stage04_Boss_Page02";
                break;

            default:
                Debug.LogError(this + "UnExpectedValue");
                break;
        }
        return result;
    }

    public string ConvertEnumToAudioClipName(SFXS _sfx)
    {
        string result = string.Empty;
        switch (_sfx)
        {
            case SFXS.GUN:
                result = "Gun";
                break;
            case SFXS.SWORD:
                result = "Sword";
                break;
            case SFXS.ARROW:
                result = "Arrow";
                break;

            default:
                Debug.LogError(this + "UnExpectedValue");
                break;
        }
        return result;
    }
    #endregion
}
```

### PoolManager 구현

이제 리소스를 불러올 준비가 되었으니, 그 리소스를 가지고  재생할 수 있도록 Pooling 시스템을 만들어 봅니다.

`Pooling` 방식은 다음과 같습니다.

1. 현재 재생중인 오디오 객체가 있는지 확인
2. 현재 재생중인 객체가 없다면, 해당 객체를 반환
3. 모든 객체가 재생중이거나, 객체가 하나도 없다면 새로운 객체를 생성



🗅 **<span style="color: #c03a92">class AudioPlayer</span>**
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(AudioSource))]
public class AudioPlayer : MonoBehaviour
{
    private AudioSource audioSource = null;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }
    public void SetClip(AudioClip _audioClip)
    {
        this.audioSource.clip = _audioClip; 
    }
    public void Play()
    {
        audioSource.Play();
    }
    public bool IsPlaying()
    {
        return audioSource.isPlaying;
    }
}
```


🗅 **<span style="color: #c03a92">class PoolManager</span>**
```cs
using System.Collections.Generic;
using System.Linq;
using Enums;
using UnityEngine;

public class PoolManager : MonoBehaviour
{
    private static PoolManager instance = null;
    public static PoolManager Instance
    {
        get
        {
            if (instance == null)
            {
                GameObject go = new GameObject("PoolManager");
                DontDestroyOnLoad(go);
                instance = go.AddComponent<PoolManager>();
            }
            return instance;
        }
    }
    private PoolManager() { }

    private List<AudioPlayer> audioPool = new List<AudioPlayer>();

    public AudioPlayer GetAudioPlayer(BGMS _bgm)
    {
        AudioPlayer result = null;
        var availablePlayer = audioPool.FirstOrDefault(player => player.IsPlaying());

        if (availablePlayer != null)
        {
            AudioClip audioClip = ResourceManager.Instance.GetAudioClip(_bgm);
            availablePlayer.SetClip(audioClip);
            result = availablePlayer;
        }
        else
        {
            GameObject prefab = ResourceManager.Instance.GetPrefab(PREFABS.AUDIO_PREFAB);
            if (prefab != null)
            {
                GameObject go = Instantiate(prefab, transform);
                AudioClip audioClip = ResourceManager.Instance.GetAudioClip(_bgm);

                AudioPlayer newAudioPlayer = go.GetComponent<AudioPlayer>();
                newAudioPlayer.SetClip(audioClip);
                audioPool.Add(newAudioPlayer);
                result = newAudioPlayer;
            }
        }
        return result;
    }

    public AudioPlayer GetAudioPlayer(SFXS _bgm)
    {
        AudioPlayer result = null;
        var availablePlayer = audioPool.FirstOrDefault(x => !x.IsPlaying());

        if (availablePlayer != null)
        {
            AudioClip audioClip = ResourceManager.Instance.GetAudioClip(_bgm);
            availablePlayer.SetClip(audioClip);
            result = availablePlayer;
        }
        else
        {
            GameObject prefab = ResourceManager.Instance.GetPrefab(PREFABS.AUDIO_PREFAB);
            if (prefab != null)
            {
                GameObject go = Instantiate(prefab, transform);
                AudioClip newAudioClip = ResourceManager.Instance.GetAudioClip(_bgm);

                AudioPlayer audioPlayer = go.GetComponent<AudioPlayer>();
                audioPlayer.SetClip(newAudioClip);
                audioPool.Add(audioPlayer);
                result = audioPlayer;
            }
        }
        return result ;
    }
}
```


### SoundManager 구현

이제 모든 준비가 되었으니, Soundmanager 를 만듭니다.

SoundManager는 다른 Manager들을 사용하여 사운드를 재생하는 함수를 가지고 있습니다.

해당하는 EnumType 에 맞게 알맞은 소리가 재생 됩니다.

🗅 **<span style="color: #c03a92">class SoundManager</span>**
```cs
using Enums;

public class SoundManager
{
    private static SoundManager instance = null;
    public static SoundManager Instance
    {
        get
        {
            if(instance == null)
            {
                instance = new SoundManager();
            }
            return instance;
        }
    }

    public void PlayBgm(BGMS _bgm)
    {
        PoolManager.Instance.GetAudioPlayer(_bgm)?.Play();
    }

    public void PlaySfx(SFXS _sfxs)
    {
        PoolManager.Instance.GetAudioPlayer(_sfxs)?.Play();
    }  
}

```


이걸 호출하는 객체는 일단 `GameManager`로 설정 하였습니다.

또한, 스테이지가 바뀌게 되면, 이전 스테이지의 리소스들은 필요가 없기 때문에 `ResourceManager`의 `ClearAudioResources()` 함수를 통해 Dictionary를 비워주고 로드를 하도록 만들었습니다.

🗅 **<span style="color: #c03a92">class GameManager</span>**
```cs
using UnityEngine;
using Enums;


public class GameManager : MonoBehaviour
{
    private StageManager stageManager = null;

    private void Awake()
    {
        stageManager = GetComponentInChildren<StageManager>();
    }

    private void Update()
    {
        //BGMs
        if (Input.GetKeyDown(KeyCode.Q))
        {
            stageManager.SetStage(STAGES.STAGE01);
            ResourceManager.Instance.ClearAudioResources();
            SoundManager.Instance.PlayBgm(BGMS.STAGE_01_START);
        }
        else if (Input.GetKeyDown(KeyCode.W))
        {
            stageManager.SetStage(STAGES.STAGE02);
            ResourceManager.Instance.ClearAudioResources();
            SoundManager.Instance.PlayBgm(BGMS.STAGE_02_START);
        }
        else if (Input.GetKeyDown(KeyCode.E))
        {
            stageManager.SetStage(STAGES.STAGE03);
            ResourceManager.Instance.ClearAudioResources();
            SoundManager.Instance.PlayBgm(BGMS.STAGE_03_START);
        }
        else if (Input.GetKeyDown(KeyCode.R))
        {
            stageManager.SetStage(STAGES.STAGE04);
            ResourceManager.Instance.ClearAudioResources();
            SoundManager.Instance.PlayBgm(BGMS.STAGE_04_BATTLE_PAGE_01);
        }

        //SFXs
        if (Input.GetKeyDown(KeyCode.A))
        {
            SoundManager.Instance.PlaySfx(SFXS.SWORD);
        }
        else if (Input.GetKeyDown(KeyCode.S))
        {
            SoundManager.Instance.PlaySfx(SFXS.GUN);
        }
        else if (Input.GetKeyDown(KeyCode.D))
        {
            SoundManager.Instance.PlaySfx(SFXS.ARROW);
        }
    }

}

```

이렇게 하면 사운드 재생이 완료 됩니다.


### 추후 개선해야 할 점

1. EnumType의 문제점

지금은 모든 소스를 EnumType 으로구성하였습니다. EnumType 으로 구현하니까, 장점은 IDE의 도움을 받아서 작성할 수 있다는 것입니다. 하지만, 단점으로는 Enum이 새로 생길 때 마다, 해당 Enum 타입을 받는 함수가 생겨야 하고, 그 뜻은 유지보수에서 힘들 수 있다는 것입니다. 

실제로 처음에는 Enum 을 구성할 때, Stage 별로 Enum을 구성하여 만들었습니다. 이렇게 하니까 `ResourceManager`의`GetAudioClip()`, `LoadAudioClip()`, `ConvertEnumToAudioClipName()` 함수를 만들 때, 해당하는 EnumType 마다의 오버로딩이 생겼습니다. 와.. 어지럽더군요.

그래서 지금은 `BGMS`, `SFXS` 이렇게 두가지의 타입으로 나누어서 만들어 놨습니다. 

확장을 고려하여 더 나은 방법이 있는지 생각 해 봐야겠습니다.


## 2024-07-08 수정

위에 문저점으로 Enums 를 제네릭화 못시킨다는 단점이 있었는데, 찾아보니까 `Enums`라는 클래스가 있었습니다.

해당 Enum 은 모든 타입의 `enum`들을 가져올 수 있습니다.

```csharp
public class ResourceManager
{
	public string ConvertEnumToAudioClipName(Enums _enum)
    {
        string result = string.Empty;
        switch (_enum)
        {
            case SFXS.GUN:
                result = "Gun";
                break;
            case SFXS.SWORD:
                result = "Sword";
                break;
            case SFXS.ARROW:
                result = "Arrow";
                break;
            case BGMS.STAGE_01_START:
                result = "BGM_Stage01";
                break;
            case BGMS.STAGE_02_START:
                result = "BGM_Stage02";
                break;
            case BGMS.STAGE_03_START:
                result = "BGM_Stage03";
                break;
            case BGMS.STAGE_03_ENTER:
                result = "BGM_Stage03_Enter";
                break;
            case BGMS.STAGE_04_BATTLE_PAGE_01:
                result = "BattleBGM1";
                break;
            case BGMS.STAGE_04_BATTLE_PAGE_02:
                result = "BattleBGM2";
                break;
            case BGMS.STAGE_04_BATTLE_PAGE_03:
                result = "BGM_Stage04_Boss_Page01";
                break;
            case BGMS.STAGE_04_BATTLE_PAGE_04:
                result = "BGM_Stage04_Boss_Page02";
                break;
            default:
                Debug.LogError(this + "UnExpectedValue");
                break;
        }
        return result;
    }
}
```

이런식으로 사용할 수 있습니다.

하지만 Enum 타입이 너무 물렁물렁해져서 예외처리를 좀 빡세게 해 줄 필요가 있어 보입니다.




