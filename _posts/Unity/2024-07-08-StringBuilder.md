---
title: "[C#] String Builder 를 사용하여 가비지 줄이기"

categories:
  -  C Sharp
  
tags:
  - [Programming, Language]

toc: true
toc_sticky: true

published: true

date: 2024-07-08
last_modified_at: 2024-07-08
---

처음 프로그래밍을 시작할 때, C 언어를 배워보신 분들이라면 `string`이라는 녀석의 정체를 알 수 있습니다.
`string`은 사실 `char[]` 라는 것 입니다.

그래서, c언어에서 문자열 합치기, 문자열 복사하기 같은것을 하면 char[] 형태의 buffer를 만들어서 문자열을 수정할 수 있었습니다.

C#은 string이라는 불변타입을 제공하고 있습니다. 그래서, string 자체에는 가비지가 생기지 않습니다. 하지만, string값을 바꾸게 되면, 내부적으로 buffer 를 만들어서 작업을 수행하게 됩니다.

이 과정에서 buffer는 한번 쓰고 버리는 것이기 때문에, 가비지가 생깁니다.

혹시나, string을 자주 바꿔야 한다면 `StringBuilder`를 사용하시는 것이 좋습니다.

`StringBuilder`는 객체를 생성하여, 전역변수로 buffer를 가지고 있기 때문에, `StringBuilder`객체가 사라지기 전까지는 buffer의 가비지가 생성되지 않습니다.

```csharp
public class SomeClass
{
	private StringBuilder someString = new StringBuilder();
	private int someInt = 100;

	private static void Main(string args[])
	{
		someString.Append(someInt);
		someString.Append("값을 출력");
	}
}
```

이런식으로 사용할 수 있습니다.

아직 string 까지 최적화 해보지는 않았지만, 매프레임 string을 바꾼다던지 한다면 `StringBuilder`가 좋은 선택지 일 것 같습니다.