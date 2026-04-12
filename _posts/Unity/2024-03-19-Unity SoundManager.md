---
layout: post

title: "[Unity] SoundPool을 활용하여 간단하게 SoundManager 구현하기"

categories:
  -  Unity
  
tags:
  - [Unity, C#, SoundManager]

toc: true
toc_sticky: true

published: true

date: 2024-03-19
last_modified_at: 2024-03-19
---

프로젝트를 하면서 사운드를 넣는 것은 필수적입니다.

사운드에는 크게 `BGM`과 `Effect Sound` 가 있습니다.

3D 사운드를 구현하기 위하여 `Sound`객체를 만들어 `SoundPool`에 저장하고, 소리가 나야 하는 곳에서 정보를 넣어주면, `SoundPool`에서 객체가 해당 위치로 이동하여 소리를 내는 방식입니다.

또한, 확장성을 위하여 `Sound`라는 모델 클래스와, `Enums`라는 네임스페이스를 만들어 매핑하는 방식을 사용하였습니다.

게임에서 `싱글톤`을 사용하는 일은 딱히 권장하지 않지만, 편의성을 위해 SoundManager를 싱글톤으로 구현하였습니다.

## Unity Sound 로직 구현


### 구조

본 Sound 로직의 구조 입니다. 단순한 하향식 구조 입니다.

![Sound 구조도](/images/Pasted%20image%2020240319162404.png)

### Sound Model 클래스 만들기

우선 사운드 모델 클래스를 만듭니다.

유니티의 `Inspector`창에 모델클래스를 붙이려면 [System.Serializable] 이라는 어노테이션을 달아야 합니다.

해당 모델클래스는 사운드의 이름과 사운드의 클립에 대한 정보를 가지고 있습니다.

🗅 **<span style="color: #c03a92">class Sound</span>**
```cs
namespace Model.Sound
{
    [System.Serializable]
    public class Sound
    {
        public Sound(string _name, AudioClip _audioClip)
        {
            this.name = _name;
            this.audioClip = _audioClip;
        }

        /// <summary>
        /// Audio 이름
        /// </summary>
        [SerializeField]
        private string name;
        public string Name { get { return name; } set { name = value; } }

        /// <summary>
        /// AudioClip
        /// </summary>
        [SerializeField]
        private AudioClip audioClip;
        public AudioClip AudioClip { get { return audioClip; } set { audioClip = value; } }

    }
}
```

### Enums 클래스 만들기

해당 namespace는 브금과 효과음의 Enum타입들을 가지고 있습니다.
```cs
namespace Enums
{
    /// <summary>
    ///<주의!> Sound 추가시 SoundManager의 배열 순서와 Enum의 배열 순서는 같아야 함
    /// </summary>
    public enum BGM
    {
        Start,
        End
    }

    /// <summary>
    ///<주의!> Sound 추가시 SoundManager의 배열 순서와 Enum의 배열 순서는 같아야 함
    /// </summary>
    public enum EFFECT_SOUND
    {
        Sword,
        Arrow,
        Gun
    }
}
```

### SoundManager 객체 및 스크립트 만들기

이제 `SoundManager`를 만들 차례 입니다. 유니티 하이어라키에 `SoundManager`를 만들고, 인스펙터에 `SoundManager.cs`파일을 등록합니다.

`SoundManager`는 브금 또는 효과음을 재생 및 정지 시키는 역할을 하고 있습니다. 
또한, `Enum`과 매핑을 시켜서 해당하는 Enum의 타입만 넣으면 Audio Clip이 재생되도록 합니다.

🗅 **<span style="color: #c03a92">class SoundManager</span>**
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Enums;
using Model.Sound;
using Unity.VisualScripting;

public class SoundManager : MonoBehaviour
{

    #region Member Variable
    #region Singleton
    private SoundManager() { }
    private static SoundManager instance;
    public static SoundManager Instance
    {
        get
        {
            if(instance == null)
            {
                return null;
            }
            else
            {
                return instance;
            }
        }
    }
    #endregion


    //audioSource 
    private AudioSource audioSource = null;

    private SoundPool soundPool = null;

    //BGM을 저장할 배열
    [SerializeField] private Sound[] BGMS = null;

    //EffectSound를 저장할 배열
    [SerializeField] private Sound[] EffectSounds = null;
    #endregion

    #region Events
    private void Awake()
    {
        #region SingleTon
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(instance);
        }
        else
        {
            Destroy(this);
        }
        #endregion

        this.audioSource = GetComponent<AudioSource>();
        this.soundPool = GetComponentInChildren<SoundPool>();
    }
    private void Start()
    {
        //시작시 브금 재생
        PlayBGM(BGM.Start);
    }
    #endregion

    #region Methods

    /// <summary>
    /// 브금 재생
    /// </summary>
    /// <param name="_bgm">재생하고 싶은 브금 Enum 주입</param>
    public void PlayBGM(BGM _bgm)
    {
        audioSource.Stop();//재생 중지
        audioSource.clip = GetBGM(_bgm);//배열의 0번째 AudioClip에 매핑
        audioSource.Play();//재생
    }

    /// <summary>
    /// 재생중인 브금 끄기
    /// </summary>
    public void StopBGM()
    {
        audioSource.Stop();
        audioSource.clip = null;
    }

    public void PlayEffectSound(Vector3 _position, EFFECT_SOUND _effectSound)
    {
        soundPool.GetObject(_position, GetEffectSound(_effectSound));
    }

    //Enum에 매핑된 AudioClip을 가져와서 재생하는 함수
    private AudioClip GetBGM(BGM _bgm)
    {
        return BGMS[(int)_bgm].AudioClip;
    }

    //Enum에 매핑된 AudioClip을 가져와서 재생하는 함수
    private AudioClip GetEffectSound(EFFECT_SOUND _effectSound)
    {
        return EffectSounds[(int)_effectSound].AudioClip;
    }
    #endregion
}

```

![SoundManager 등록](/images/Pasted%20image%2020240319160234.png)

등록한 후, Sound 들의 이름과 AudioClip을 매핑시켜 줍니다.

*<주의!> Enum의 순서와 AudioClip의 순서는 항상 일치해야 합니다.*


### 사운드 풀 만들기

...풀 이라는 기술은 여러 오브젝트들을 관리하기 위하여 사용하는 기술 입니다.

저는 사운드 객체를 여러개 만들 생각이므로, `SoundPool`을 구현 합니다.

물론 더 좋은 라이브러리가 있지만, 제가 커스텀으로 만든 간단한 `SoundPool` 입니다.

해당 클래스는 단순히 `Start()` 함수 호출 시, `SoundPrefab`을 생성해줍니다.
또한, `SoundPrefab`에서 음악 재생이 끝나면, 다시 꺼주는 역할을 합니다. 

🗅 **<span style="color: #c03a92">class SoundPool</span>**
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SoundPool : MonoBehaviour
{
    #region Member Variables
    [SerializeField] GameObject prefab;
    [SerializeField] private int poolSize = 10;

    private List<GameObject> objects;
    #endregion

    #region Events
    private void Awake()
    {
        objects = new List<GameObject>();
    }        

    private void Start()
    {
        //SoundPool 생성
        CreateSoundPools();
    }
    #endregion

    #region Methods

    public GameObject GetObject(Vector3 position, AudioClip audioClip)
    {
        foreach (var obj in objects)
        {
            if (!obj.activeInHierarchy)
            {
                obj.SetActive(true);
                obj.transform.position = position;
                obj.GetComponent<SoundPrefab>().PlayAudioClip(audioClip);
                return obj;
            }
        }

        return null;
    }

    public void ReturnObjectToPool(SoundPrefab obj)
    {
        obj.gameObject.SetActive(false);
    }

    private void CreateSoundPools()
    {
        for (int i = 0; i < poolSize; i++)
        {
            GameObject obj = Instantiate(prefab);
            obj.SetActive(false);
            obj.GetComponent<SoundPrefab>().SetOnPlayEndCallback(ReturnObjectToPool);
            obj.transform.SetParent(this.transform);
            objects.Add(obj);
        }
    }
    #endregion
}

```

### 사운드 프리팹 만들기

이제 사운드를 재생해줄 사운드 객체를 만듭니다.

`SoundPrefab` 에서는 `OnEable()` 에서 효과음을 한번만 재생하도록 합니다. 

그리고 콜백을 만들어서 해당 AudioClip재생이 끝나면, SoundPool의 `ReturnToObjectPool()` 함수를 호출하여 다시 꺼주도록 합니다.

여기서 콜백을 만든 이유는 오디오의 재생이 끝났음을 알려주는 프로퍼티를 못찾아서 입니다. 더 좋은 방법있으면 알려주세용

🗅 **<span style="color: #c03a92">class SoundPrefab</span>**
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SoundPrefab : MonoBehaviour
{
    #region Member Variables
    //효과음 재생이 끝났음을 알리기 위한 콜백
    public delegate void VoidSoundPrefabDelegate(SoundPrefab _soundPrefab);
    private VoidSoundPrefabDelegate onPlayEndCallback = null;

    //오디오 소스 관련 변수
    private AudioSource audioSource = null;
    private AudioClip audioClip = null;
    #endregion

    #region Events
    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }
    private void OnDisable()
    {
        //음악 재생이 끝나면 SoundPool 에게 콜백을 넘김
        onPlayEndCallback?.Invoke(this);
    }
    private void Update()
    {
        Debug.Log(this.audioSource.isPlaying);
    }
    #endregion

    #region Events

    //음악이 재생되고 있는지 체크하는 코루틴
    private IEnumerator CheckAudioPlay()
    {
        while (true)
        {
            if (!audioSource.isPlaying)
            {
                this.gameObject.SetActive(false);
                yield return null;
            }
            yield return null;
        }
    }

    public void PlayAudioClip(AudioClip _audioClip)
    {
        this.audioClip = _audioClip;
        //audioSource 재생
        this.audioSource.PlayOneShot(this.audioClip);

        //음악이 재생되고 있는지 확인하는 코루틴 실행
        StartCoroutine(CheckAudioPlay());
    }

    /// <summary>
    /// SoundPrefab 에서 음악 재생 끝났음을 알리는 이벤트
    /// </summary>
    /// <param name="_onPlayEndCallback">인자로 SoundPrefab 전달</param>
    public void SetOnPlayEndCallback(VoidSoundPrefabDelegate _onPlayEndCallback)
    {
        this.onPlayEndCallback = _onPlayEndCallback;
    }
    #endregion
}

```

## 실행

이제 다 만들었습니다. 만든 로직을 호출해 봅시다.

### BGM
우선 BGM 입니다.

BGM은 게임 전체적으로 수행되어야 하기 때문에, `GameManager`가 관리하도록 합니다.
게임 시작 시, GameStart 브금을 재생해주는 것을 구현해 봅니다. 

🗅 **<span style="color: #c03a92">class GameManager</span>**
```cs
using UnityEngine;
using Common.Method;
using System;
using UnityEngine.Networking;
using Enums;
using System.Collections;

public class GameManager : MonoBehaviour
{
    #region Manager
    private SoundManager soundManager = null;
    private LogManager log = null;
    
    #endregion

    public bool isGameOver = false;
    
    #region SingleTon
    private GameManager() { }
    private static GameManager instance = null;

    public static GameManager Instance
    {
        get
        {
            if (instance == null)
            {
                return null;
            }
            else
            {
                return instance;
            }
        }
    }
    #endregion

    #region 이벤트
    private void Awake()
    {
        //객체등록
        #region SingleTon
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(this.gameObject);
        }
        else
        {
            Destroy(this.gameObject);
        }

        #endregion
        soundManager = GetComponentInChildren<SoundManager>();

        log = new LogManager("GameManager");
    }

    private void Start()
    {
	    //브금 호출
		soundManager.PlayBGM(BGM.Start);
    } 
    
    #endregion  
    
}

```

이런식으로 구현하게 됩니다.


### Effect Sound

이제 효과음 입니다.

무기에서 효과음을 재생해보도록 합니다.

🗅 **<span style="color: #c03a92">class Arrow</span>**
```cs
using Enums;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Arrow : Weapon
{

    #region Member Variables


    private SoundManager soundManager;

    #endregion

    #region Events

    private void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
        soundManager  = SoundManager.Instance;  
    }

    private void Update()
    {
        Attack();
    }
    #endregion

    #region Methods

    protected override void Attack()
    {
        Vector2 originPos = this.transform.parent.GetComponent<WeaponPos>().transform.position;

        if (Input.GetKeyDown(KeyCode.Mouse1))
        {
            if (isAttack) return;
            isAttack = true;

			//공격로직 생략
			
            soundManager.PlayEffectSound(this.transform.position, EFFECT_SOUND.Arrow);//effectsound재생
        }

    }

    public override float GetWeaponDmg()
    {
        return attackDmg;
    }
    #endregion

}

```

이런식으로 구현하면 됩니다.

일단 이렇게 사용하다가, 조금씩 살을 더 붙여나갈 계획입니다. 

부족한 부분이나 고쳐야할 부분이 있다면 언제든지 지적해주세요!