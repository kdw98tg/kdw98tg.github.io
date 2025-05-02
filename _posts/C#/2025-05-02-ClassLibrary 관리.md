---
title:  "[C#] C# ClassLibrary로 재사용 가능한 모듈 만들기" 

categories:
  -  C Sharp
tags:
  - [Programming, Language, ClassLibrary]

toc: true
toc_sticky: true

published: true

date: 2025-05-02
last_modified_at: 2025-05-02
---

프로그램을 제작할 때에는 공용으로 필요한 기능들이 있습니다. 예를들어 더하기, 뺴기, 자료형 파싱하기 등의 기능입니다. 해당 기능들은 부수효과를 일으키지 않고, 프로젝트를 할때 자주 필요한 기능들입니다.

이러한 기능들을 매 프로젝트를 할 때 마다 작성하는것은 시간이 너무 많이 드는 작업입니다. 따라서 해당 기능들을 배포가능한 컴포넌트(dll)로 만들어서 관리하는것이 좋습니다. C#에서는 dll을 만들기 위해 ClassLibrary를 만들어 사용합니다.

## 클래스 라이브러리 만드는 법

우선, Visual Studio를 켠 다음, Console App 을 선택하여 프로젝트를 만듭니다. ConsoleApp은 Class Library를 만든 후, 실행해보기 위한 용도입니다.

![](/images/Pasted%20image%2020250502110929.png)

프로젝트를 만들면, 다음과 같은 계층구조로 이루어진것을 볼 수 있습니다.

![](/images/Pasted%20image%2020250502111126.png)

여기서 `Solution`을 선택한 후, 우클릭 하여 Add - New Project 를 선택합니다. 그리고 ClassLibrary를 생성해 줍니다.

![](/images/Pasted%20image%2020250502111307.png)

저는 이름을 `MathUtil`로 지정하였습니다. 클래스 라이브러리를 만들면 `Solution`안에 ClassLibrary가 하나 더 생긴것을 볼 수 있습니다.

![](/images/Pasted%20image%2020250502111354.png)

이제, MathUtil 이라는 ClassLibrary 안에 있는 Class1을 MathUtil.cs 로 바꾼 다음에 함수를 하나 작성합니다. 함수는 간단하게 Add, Sub 함수 2개만 작성합니다.


![](/images/Pasted%20image%2020250502111525.png)

## 테스트하기

이제, ClassLibrary에 함수들을 작성하였으니, 이전에 만들어두었던 ConsoleApp 을 통하여 테스트 해봅니다. 그러기 위해서는 ConsoleApp에 방금 만든 MathUtil 라이브러리의 참조를 추가해줘야 합니다.

ConsoleApp의 라이브러리를 선택하고, 우클릭 - Add - Project Reference 를 해줍니다. 그러면 방금만든 MathUtil 라이브러리가 뜨게 됩니다.

![](/images/Pasted%20image%2020250502111732.png)

MathUtil을 선택하고 Ok를 눌러줍니다. 

이제, Program.cs 로 와서, 네임스페이스에 MathUtil을 추가해주고 함수를 사용해 봅니다.

![](/images/Pasted%20image%2020250502112059.png)

정상적으로 잘 작동하는것을 볼 수 있습니다.

이제, 해당 라이브러리를 dll로 빌드하여 재사용가능한 모듈로 만들어야 합니다. `MathUtil`프로젝트를 빌드하고, 프로젝트 경로에서 dll을 가져와 다른 프로젝트에 넣으면 됩니다.

여기서 주의해야 할 점이, 빌드를 하기 전에 우측에 있는 빌드 모드를 `Debug`에서 `Release`로 바꿔줘야 합니다.

![](/images/Pasted%20image%2020250502112443.png)

Release 모드로 바꾸고 나서 빌드를 하고, 프로젝트의 경로로 가면 bin 폴더 아래에 Release 폴더가 생성됩니다. Release 폴더 안으로 들어가게 되면, dll이 생성된것을 볼 수 있습니다.

![](/images/Pasted%20image%2020250502112610.png)