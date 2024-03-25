---
title: "[Unity] UnityWebRequest를 사용한 통신 로직 구현"

categories:
  -  Unity
  
tags:
  - [Unity, C#, UnityWebRequest, Coroutine]

toc: true
toc_sticky: true

published: true

date: 2024-03-25
last_modified_at: 2024-03-25
---


상용화가 목적인 프로젝트에서 클라이언트와 서버가 통신을 하는 것은 필수적입니다.

통신에는 여러종류가 있지만, 이번에 구현한 통신방식은 HTTP 통신 입니다.

`Winform`을 할 때는 `MySQL.dll`를 임포트하여 `MySQLConnection`을 사용하여 구현했습니다. 하지만, 게임에서는 통신중에 다른 로직이 멈추면 안되기에 비동기 처리를 권장하고 있었습니다. 그리하여 
`UnityWebRequest`와 `Coroutine`을 사용하여 비동기로 구현하였습니다.

또한, `Newtonsoft.Json`패키지를 사용하였습니다. 해당 패키지는 Window - PackageManager 에서 이름으로 검색을 선택 한 후, `com.unity.nuget.newtonsoft-json` 을 입력하면 됩니다.

![package Manager](/images/Pasted%20image%2020240325132253.png)

## Model 클래스 만들기

본 예제는 MVC 패턴을 기준으로 작성되었습니다. 우선 데이터를 담을 Model 클래스를 만듭니다.

경로는 Asset/Model/User 입니다.

🗅 **<span style="color: #c03a92">class UserModel</span>**
```cs
using Newtonsoft.Json;

namespace Model.User
{
    [System.Serializable]
    public class UserModel
    {
        [JsonProperty("user_code")]
        public string UserCode;

        [JsonProperty("user_nick")]
        public string UserNick;
    }
}
```

그리고 서버에서 results 라는 키값을 넣은 json배열을 보낼것이기 때문에 이 `results` 를 담을 수 있는 제네릭 모델을 만들어 줍니다.

경로는 Asset/Model/Result 입니다.

🗅 **<span style="color: #c03a92">class ResultModel</span>**
```cs
namespace Model.Result
{
    [System.Serializable]
    public class ResultModel<T>
    {
        public T[] results;
    }
}
```

## Repository 클래스 만들기

이제 직접 데이터베이스에 접근할 로직을 구현하기 위해 `Repository` 클래스를 구현 합니다.
(이름은 생각나는대로 했는데 혹시 더 좋은 이름이 있다면 추천해주세요. )
또한, User 정보에 접근하기 위한 서버 주소도 해당 repository에서 관리하게 됩니다.

경로는 Asset/Script/DataAccess/User 입니다.

여기서 `CheckNull`함수는 디비에서 넘어올 때 값이 `NULL`이면 빈 문자열을 반환하는 메서드 입니다.

아래에는 `DeserializeObject()`를 사용하여 클래에 바로매핑하는 방식과, `JsonObject` 를 받아서 해체하는 방식 두가지로 작성하였습니다.

🗅 **<span style="color: #c03a92">class UserRepository</span>**
```cs
using Model.Result;
using Model.User;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Collections;
using UnityEngine;
using UnityEngine.Networking;
using UnityEngine.UIElements;
using Common.Method;

namespace DataAccess.User
{

    public class UserRepository
    {
        private CommonMethod method = new CommonMethod();

        private string url = "https://successlist.mycafe24.com/successlist/php/SELECT_USER.php";

        public IEnumerator SELECT_USER(string _userCode, Action<bool, UserModel> _action)
        {
            WWWForm form = new WWWForm();
            form.AddField("userCode", _userCode);

            using (UnityWebRequest request = UnityWebRequest.Post(url, form))
            {
                yield return request.SendWebRequest();

                if (request.result == UnityWebRequest.Result.ConnectionError || request.result == UnityWebRequest.Result.ProtocolError)
                {
                    _action(false, null);
                    //로깅작업 생략
                }
                else
                {
                    try
                    {
                        //결과 json
                        JObject jsonResponse = JObject.Parse(request.downloadHandler.text);

                        //하나씩 뜯는 경우
                        JArray resultsArray = (JArray)jsonResponse["results"];

                        UserModel model = new UserModel();

                        string userCode = method.CheckNull(resultsArray[0]["user_code"].ToString());
                        string nickName = method.CheckNull(resultsArray[0]["user_nick"].ToString());

                        model.UserCode = userCode;
                        model.UserNick = nickName;

                        //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

                        //클래스에 바로 대입하는 경우
                        string jsonResponse1 = request.downloadHandler.text;
                        ResultModel<UserModel> jsonResult = JsonConvert.DeserializeObject<ResultModel<UserModel>>(jsonResponse1);

                        // 결과 확인
                        UserModel userModel = new UserModel();


                        //json 배열로 들어왔을 때 foreach 사용
                        foreach (var user in jsonResult.results)
                        {
                            userModel.UserCode = user.UserCode;
                            userModel.UserNick = user.UserNick;
                        }

                        _action(true, model);
                    }
                    catch (Exception ex)
                    {
                        _action(false, null);
                        //로깅작업 생략
                    }
                }
            }
        }

        public IEnumerator INSERT_USER() { yield return null; }
        public IEnumerator UPDATE_USER() { yield return null; }
        public IEnumerator DELETE_USER() { yield return null; }
    }
}

```


해당 Repository는 데이터 베이스에 접근하여, 정상적으로 값을 받아온 경우에는 `true`를, 값을 받아오지 못한 경우에는 `false`를 반환하게 됩니다.

또한 UserModel에 정보를 넣어서 Action에서 반환하도록 해줍니다.

## 실행부

이제 준비가 끝났으니, 실행부에서 실행을 해봅니다.

실행부에서는 통신이 필요할 때, 해당 Repository의 함수를 참조하도록 합니다.

🗅 **<span style="color: #c03a92">class ConnectionTest</span>**
```cs
using System;
using System.Collections;
using UnityEngine;
using DataAccess.User;

public class ConnectionTest : MonoBehaviour
{
    private void Start()
    {
        UserRepository userRepository = new UserRepository();
        StartCoroutine(userRepository.SELECT_USER("tICzO", (success, userModel) =>
        {              
            if (success)
            {
                SomeAction(userModel.UserCode, userModel.UserNick);
            }
            else
            {
                Debug.Log("네트워크 연결 실패");
            }
        }));
    }

    private void SomeAction(string _userCode, string _userNick)
    {
        Debug.Log(_userCode);
        Debug.Log(_userNick);
    }
}

```

여기서 코루틴에서 실행되며 반환된 success 값을 기준으로 통신에 성공하였을 때, 실패하였을 때 처리할 로직을 나누어서 구현하도록 합니다.

혹시 잘못된 부분이나, 수정할 부분이 있다면, 언제든지 댓글로 피드백 부탁드립니다!
