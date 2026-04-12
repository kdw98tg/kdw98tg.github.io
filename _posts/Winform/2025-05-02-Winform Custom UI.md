---
title: "[Winform] Winform Custom UI 만들기"

categories:
  -  Winform
  
tags:
  - [Winform, C#, UI]

toc: true
toc_sticky: true

published: true

date: 2025-05-02
last_modified_at: 2025-05-02
---

Winform은 2000년 7월 첫 릴리즈 되었습니다. 즉, 2025년인 지금부터 약 25년전 프레임워크 인것이죠. Designer가 C#으로 이루어진 코드로 되어있어서 커스텀하기가 참 불편합니다. 그리고 기본 제공되는 UI들의 디자인도 모두 구립니다.. 이러한 점을 개선하여 xaml로 Design 할 수 있도록 나온 WPF가 나오긴 했습니다만, Winform의 단순한 프레임워크 구조는 무시할 수 없기 때문에, Winform 에서도 방법을 찾아야 합니다.

그래서 Nuget Package를 보면, 유명한 커스텀 UI 패키지인 Bunifu 라이브러리 같은것이 인기입니다. 하지만, 돈을 주고 써야하는 단점이 있습니다.

그때 생각이 든 것이, 프로젝트를 할때마다 커스텀 UI들을 생성하여 나만의 커스텀 UI 패키지를 만들어야 겠다라는 것이었습니다.

또한, Winform 이 기본제공하는 UI들을 한번 더 래핑해서 캡슐화 하여 사용하면 더 좋겠다는 생각을 했습니다.

우선 예시로, 커스텀 버튼을 만들어 보겠습니다. 방법은 간단합니다. 윈폼에서 기본 제공하는 Button을 상속받은 클래스를 만들면 됩니다.

## 버튼 클래스 래핑하기

```cs
using System.Windows.Forms;

namespace CustomUI.UIs
{
    public class ButtonBase : Button
    {

    }
}
```

참으로 간단합니다. 이제 여기서 제가 사용할 함수들을 랩핑하도록 하겠습니다. 일단 ForeGround를 바꾸는 기능만 캡슐화 합니다.

```cs
using System.Drawing;
using System.Windows.Forms;

namespace CustomUI.UIs
{
    public class ButtonBase : Button
    {
        public void SetForeColor(Color _color)
        {
            this.ForeColor = _color;
        }
    }
}
```

이렇게 하면, ForeColor를 구현하는 행위 자체가 캡슐화 되고, 다른 특정 버튼을 만들때, 해당 함수를 오버라이딩 하던지 해서 기능을 확장할 수 있습니다. 가령 모서리가 둥근 버튼이라던지, 모서리가 없는 버튼이라던지 등등을 만들 수 있는 것입니다.

또한, 코드를 함수로 캡슐화 하면서 얻을 수 있는 이점은, 이 함수가 어떤 기능을 하는지 한눈에 알아볼 수 있다는 점입니다.

또 강력한 이점은, 프레임워크의 Button 클래스가 바뀌게 되면 ButtonBase의 클래스만 변경해주면 되기 때문에 유지보수에도 강력한 이점을 지닙니다.

Form에 CustomButton을 넣는 과정은 일반 Button과 같습니다.

![](/images/Pasted%20image%2020250502141303.png)

![](/images/Pasted%20image%2020250502141321.png)

그리고, Button 클래스를 상속받았기 때문에 제가 조절할 수 있는 프로퍼티는 일반 Button 클래스와 같고, 확장을 한다면 더 많아질 수 있게 되는것 입니다.

얼핏 현업에서 모든 자료형은 한번 랩핑 되어 사용된다고 들었습니다. 가령 int 같은 자료형도 Int 라는 클래스로 감싸서 사용하는것이 그 예 입니다. 이것을 깨닫고 나서 유지보수가 필요한 기능은 전부 커스텀으로 랩핑하여 사용하게 되었습니다.
