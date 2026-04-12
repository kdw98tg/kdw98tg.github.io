---
title: "[Unity] Unity Image 업로드/다운로드 (Upload/Download) 하기"

categories:
  -  Unity
  
tags:
  - [Unity, Image, Upload, Download]

toc: true
toc_sticky: true

published: true

date: 2024-10-14
last_modified_at: 2024-10-14
---

유니티에서 게임을 만들다 보면, 이미지를 업로드, 다운로드 해야하는 상황이 생깁니다. 일반적으로 프로필 사진 같은게 있습니다.

이미지를 저장할 때는, DB에 바로 이미지를 넣는게 아니라, 서버에 이미지를 저장하고, 그 서버의 디렉토리를 DB에 저장하여 불러오는 식으로 합니다. 

물론 DB에 이미지를 바이트로 변환해서 저장하는 방식이 있습니다. 하지만 속도가 엄청나게 느리기 때문에 권장하는 방법은 아닙니다.

API는 `ASP.net core`를 사용하였습니다. 예전에 `php`를 써서 업로드 할 때는 서버 루트 디렉토리 아래에 디렉토리를 만들어서 저장했었는데, `Asp.net core` 는 해당 솔루션이 있는 폴더에 `wwwroot`라는 파일을 생성해서 여기에 저장하는 방식을 사용합니다.


## 이미지 업로드

### Texture2D 형태와 Sprite 형태에서 변환하기

우선, Unity 에서 서버로 이미지를 올리는 코드를 작성 합니다. 유니티에서는 `Sprite` 라는 형태로 이미지를 사용하지만, 서버에 올리때는 `Texture2D` 형태로 바꾸어서 올리게 됩니다.

이 과정 (Sprite -> Texture2D , Texture2D -> Sprite)을 해주는 함수를 정의 합니다.

```csharp
public static class UnityMethod
{
	public static Sprite Texture2DToSprite(Texture2D _texture)
	{
	return Sprite.Create(_texture, new Rect(0, 0, _texture.width, _texture.height), new Vector2(0.5f, 0.5f));
	}
	
	public static Texture2D SpriteToTexture2D(Sprite sprite)
	{
	// Sprite의 텍스처 영역을 가져옴
	var rect = sprite.rect;
	var texture = new Texture2D((int)rect.width, (int)rect.height);
	
	// Sprite의 픽셀 데이터를 새 텍스처에 복사
	var pixels = sprite.texture.GetPixels((int)rect.x, (int)rect.y, (int)rect.width, (int)rect.height);
	
	texture.SetPixels(pixels);
	texture.Apply();
	
	return texture;
	}
}
```

### ImageController 생성

이제 변환하는 코드를 만들었으니, 실제로 서버에 업로드 하는 코드를 생성 합니다. 유니티에서는 통신을 비동기로 하는것을 권장하기 때문에 코루틴을 사용하여 통신을 합니다.

우선, Upload 하는 코드부터 작성합니다.

```csharp
using UnityEngine;
using UnityEngine.Networking;
using System.Collections;
using Common.Define;
using UnityEditor;
using System;

public class ImageController
{
    private string imageUploadURL = Define.ServerURL + "/api/Image/upload";

    public IEnumerator UploadImage(string _fileName, Texture2D _image, Action<bool, string> _action)
    {
        byte[] imageBytes = _image.EncodeToPNG();

        WWWForm form = new WWWForm();

        //AddbinaryData 의 첫번째 매개변수 이름이 서버의 매개변수 이름과 다르면 ProtocolError 를 뿜음
        //imageByte 는 사진이 바이트로 바뀐 형태
        //3번쨰는 이미지 이름
        //4번째는 이미지 형식
        form.AddBinaryData("_file", imageBytes, _fileName, "image/png");

        using (UnityWebRequest request = UnityWebRequest.Post(imageUploadURL, form))
        {
            yield return request.SendWebRequest();

            if (request.result == UnityWebRequest.Result.ConnectionError ||
                request.result == UnityWebRequest.Result.ProtocolError)
            {
                Debug.LogError("Image upload failed: " + request.error);
                _action(false, request.error);
            }
            else
            {
                // 서버 응답 처리
                string responseText = request.downloadHandler.text;
                _action.Invoke(true, responseText);
            }
        }
    }
}


```


이 코드는 이미지를 받아서, 바이트코드로 변환 한 후, `WWWForm` 객체에 실어서 서버에 바이트 코드를 보내는 역할을 합니다.

여기서 주의깊게 봐야할 곳은 `form.AddBinaryData` 입니다.

WWWForm.AddBinaryData 는 4가지의 인자를 가집니다. (필드명, 바이트코드, 파일명, 파일 형식) 으로 구성되어 있습니다.

여기서, 필드명을 `ASP.net` 에 있는 필드명과 같게 만들어주어야 합니다. 만약, 이것이 같지 않으면 `ProtocolError`를 뿜게 됩니다.

다음은, 해당 코드를 사용하는 방법 입니다.

```csharp
    private void OnProfileButtonClick()
    {
        ImageController imageController = new ImageController();
        
        //upload
        StartCoroutine(imageController.UploadImage(CommonMethod.GetUserProfileImageName(Global.Instance.User.User_Code), UnityMethod.SpriteToTexture2D(buttonProfile.GetImage()), (onSuccess, message) =>
        {
           if (onSuccess)
           {
               Debug.Log(message);
           }
        }));
    }
```

위 코드에서 파일명과, Sprite를 Texture 로 바꿔주는 과정을 수행 합니다. 그리고, 업로드가 완료되면 콜백으로 성공 여부와 메세지를 넘겨주게 됩니다.


### 서버 코드

이제 서버코드를 살펴 봅니다. 처음에 `IWebHostEnvironment` 라는 변수를 DI 받게 되는데 이것은 애플리케이션이 실행중인 웹 호스팅 환경에 대한 정보를 제공하는 클래스 입니다.,

또한, ASP 에서는 이미지를 저장할 때 `wwwroot`라는 폴더에 저장하게 됩니다. `MVC WEB`을 만들면 이 디렉토리가 자동으로 생성 되지만, `WebAPI`를 만들면 자동으로 생성이 안됩니다. 이것을 솔루션이 있는 디렉토리에 직접 생성 해줘야 합니다.

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.VisualBasic;
using System.IO;
using System.Threading.Tasks;

[ApiController]
[Route("api/[controller]")]
public class ImageController : ControllerBase
{
    //애플리케이션이 실행 중인 웹 호스팅 환경에 대한 정보를 제공함
    //애플리케이션의 환경변수, 루트 경로 등을 받음
    private readonly IWebHostEnvironment environment;
    public ImageController(IWebHostEnvironment _environment)
    {
        environment = _environment;
    }

    [HttpPost("upload")]
    public async Task<IActionResult> Upload(IFormFile _file)
    {
        if (_file == null || _file.Length == 0)
        {
            return BadRequest("File is not selected");
        }

        //environment.WebRootPath 는 웹 애플리케이션의 기본 디렉토리 경로를 제공함
        //여기에 uploads 폴더를 추가하여 저장될 디렉토리를 만듬
        //wwwroot/uploads/ 이렇게 저장됨
        //WebApiController 에서는 wwwroot 디렉토리가 없기 때문에, 만들어줘야함. 아니면 ArgumentNullException 이 뜸
        string path = Path.Combine(environment.WebRootPath, "uploads");

        //파일을 저장할 경로가 존재하는지 확인해보고, 없으면 경로를 만듦
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }

        string fileName = Guid.NewGuid().ToString() + Path.GetExtension(_file.FileName);
        string filePath = Path.Combine(path, fileName);

        using (var stream = new FileStream(filePath, FileMode.Create))
        {
            await _file.CopyToAsync(stream);
        }
        return Ok(new { fileName });
    }
}
```

이렇게 하면, `/wwwroot/uploads` 경로에 이미지가 들어가는것을 볼 수 있습니다.

![](/images/Pasted%20image%2020241015000229.png)

여기서, 제가 지정한 이름이 아니라 알아볼 수 없는 문자가 들어가는 것은 중복방지를 위해 UUID 를 생성하는 코드를 썼기 때문입니다. 

```csharp
string fileName = Guid.NewGuid().ToString() + Path.GetExtension(_file.FileName);
```

도메인에서 중복 불가능한 형식을 이미 사용하고 있다면 해당 코드는 쓸 필요가 없습니다.
## 이미지 다운로드

이제 업로드를 했으니, 다운로드 하는 과정을 만들어 봅니다. 전체적인 과정은 업로드와 동일 합니다.

### ImageController 

이제 ImageController 에서 download 를 하는 코드를 만듭니다.

```csharp
using UnityEngine;
using UnityEngine.Networking;
using System.Collections;
using Common.Define;
using UnityEditor;
using System;

public class ImageController
{
    private string imageDownloadURL = Define.ServerURL + "/api/Image/download";

    public IEnumerator DownloadImage(string _fileName, Action<bool, Texture2D> _action)
    {
        using (UnityWebRequest request = UnityWebRequestTexture.GetTexture(imageDownloadURL + "/" + _fileName))
        {
            yield return request.SendWebRequest();

            if (request.result == UnityWebRequest.Result.ConnectionError ||
                request.result == UnityWebRequest.Result.ProtocolError)
            {
                Debug.LogError("Error downloading image: " + request.error);
                _action(false, null);
            }
            else
            {
                // 다운로드한 텍스처를 가져옵니다.
                Texture2D texture = DownloadHandlerTexture.GetContent(request);
                _action?.Invoke(true, texture);
            }
        }
    }
}
```

해당 코드는 서버에서 파일 이름을 기준으로 이미지를 조회하여 가져오는 코드 입니다. 그리고 콜백을 통해서, 성공 여부와 서버에서 받은 이미지를 넘겨 줍니다.

이 코드를 사용하는 부분은 다음과 같습니다.

```csharp
    private void OnProfileButtonClick()
    {
        ImageController imageController = new ImageController();
        StartCoroutine(imageController.DownloadImage(CommonMethod.GetUserProfileImageName(Global.Instance.User.User_Code), (onSuccess, texture) =>
        {
            buttonProfile.SetImage(UnityMethod.Texture2DToSprite(texture));
        }));
    }
```

### 서버 코드

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.VisualBasic;
using System.IO;
using System.Threading.Tasks;

[ApiController]
[Route("api/[controller]")]
public class ImageController : ControllerBase
{
    //애플리케이션이 실행 중인 웹 호스팅 환경에 대한 정보를 제공함
    //애플리케이션의 환경변수, 루트 경로 등을 받음
    private readonly IWebHostEnvironment environment;
    public ImageController(IWebHostEnvironment _environment)
    {
        environment = _environment;
    }
    
    [HttpGet("download/{fileName}")]
    public IActionResult DownloadImage(string fileName)
    {
        // 이미지 파일이 저장된 폴더 경로
        string filePath = Path.Combine(environment.WebRootPath, "uploads", fileName);

        // 파일이 존재하는지 확인
        if (!System.IO.File.Exists(filePath))
        {
            return NotFound("Image not found");
        }

        // 파일을 바이트 배열로 읽어들입니다.
        byte[] imageData = System.IO.File.ReadAllBytes(filePath);

        // 파일의 Content-Type을 image/png로 설정하고 반환합니다.
        return File(imageData, "image/png");
    }
}
```

위 코드는 `wwwroot/uploads`  디렉토리에서 파일 명으로 이미지를 찾은 다음에, 바이트 코드로 변환해서 전송해주는 코드 입니다.

코드를 실행시키면, 잘 동작하는것을 볼 수 있습니다.

![](/images/Pasted%20image%2020241015001915.png)





