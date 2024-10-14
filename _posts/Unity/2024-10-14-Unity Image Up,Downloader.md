---
title: "[Unity] Unity Image 업로드/다운로드 (Upload/Download) 하기"

categories:
  -  Unity
  
tags:
  - [Unity, Image, Upload, Download]

toc: true
toc_sticky: true

published: false

date: 2024-10-14
last_modified_at: 2024-10-14
---

유니티에서 게임을 만들다 보면, 이미지를 업로드, 다운로드 해야하는 상황이 생깁니다. 일반적으로 프로필 사진 같은게 있습니다.

이미지를 저장할 때는, DB에 바로 이미지를 넣는게 아니라, 서버에 이미지를 저장하고, 그 서버의 디렉토리를 DB에 저장하여 불러오는 식으로 합니다. 

물론 DB에 이미지를 바이트로 변환해서 저장하는 방식이 있습니다. 하지만 속도가 엄청나게 느리기 때문에 권장하는 방법은 아닙니다.

API는 `ASP.net core`를 사용하였습니다. 예전에 `php`를 써서 업로드 할 때는 서버 루트 디렉토리 아래에 디렉토리를 만들어서 저장했었는데, `Asp.net core` 는 해당 솔루션이 있는 폴더에 `wwwroot`라는 파일을 생성해서 여기에 저장하는 방식을 사용합니다.


## Unity 코드 작성

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

private string imageDownloadURL = Define.ServerURL + "/api/Image/download";

  

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

_action(false, null);

}

else

{

// 서버 응답 처리

string responseText = request.downloadHandler.text;

_action.Invoke(true, responseText);

}

}

}

  

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





