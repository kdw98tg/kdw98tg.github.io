---
title: "[Clean Architecture] 프레젠터와 험블 객체"

categories:
  -  Clean Architecture
  
tags:
  - [Clean Architecture]

toc: true
toc_sticky: true

published: true

date: 2025-05-25
last_modified_at: 2025-05-25
---

## 험블 객체 패턴

험블 객체 패턴은 디자인 패턴으로, 테스트하기 어려운 행위와 테스트하기 쉬운 행위를 단위 테스트 작성자가 분리하기 쉽게 하는 방법입니다.

이 패턴이 나오게된 아이디어는 굉장히 단순합니다. 기본적인 본질은 남기고, 테스트 하기 어려운 행위를 험블 객체로 옮기는 것입니다.

험블(Humble)은 한국어로 번역하면 `남들보다 중요하지 않은`이라는 뜻입니다. 처음에 이것만 볼떄는 무슨뜻인지 몰라서 예제를 찾아보며 이해를 했습니다.

험블객체의 예로는 UI 와 특정로직의 분리로 볼 수 있습니다. UI는 대개 테스트하기가 어려운 클래스 입니다. UI 클래스에 특정 로직이 들어가있다면, 더욱 테스트하기 어려워집니다.

예로 가져온것은 `LoginUI`라는 View 코드에 담겨있는 `Login`로직 입니다.

### 잘못된 예

험블객체 패턴을 사용하지 않고, 그냥 코드를 작성한다면 다음과 같은 모양이 나옵니다.

```cs
namespace CleanArchitecture.src.HumblePattern.Incorrect
{
    public class LoginUI
    {
        private readonly TextView textView;

        public LoginUI()
        {
            textView = new TextView();
        }

        private void OnLoginButtonClicked()
        {
            if (textView.Text == "123")
            {
                RequestLogin();
            }
            else
            {
                MessageView messageView = new MessageView();
                messageView.Show("로그인 실패!");
            }
        }

        private void RequestLogin()
        {
            //Login Logic
            bool onSuccess = false;
            if (onSuccess)
            {
                MessageView messageView = new MessageView();
                messageView.Show("로그인 성공!");
            }
        }
    }
}
```

UI코드와 로그인 객체가 섞여있는 모양입니다. 이래서는 특정 모듈을(여기서는 Login 모듈) 테스트하기 어렵습니다. 테스트하기 어려운 UI 코드가 엮여있기 때문입니다.

이것을 험블객체 패턴을 사용하여 분리합니다.

### 옳게된 예

이제 UI 로직에서 Login 로직을 분리하여 `LoginService`라는 컴포넌트에 캡슐화 합니다.

```cs
namespace CleanArchitecture.src.HumblePattern.Correct
{
    public class LoginService
    {
        public bool OnLogin(string _password)
        {
            if (_password == "123")
                return true;
            else
                return false;
        }
    }
}
```

실제 어플리케이션의 로직은 더 복잡하겠지만, 지금은 그냥 `123`이라는 비밀번호와 인자로 들어온값이 일치하면 true를 반환하도록 합니다.

이제, UI 코드에서 LoginService를 생성하여, OnLogin 함수를 호출합니다.

```cs
namespace CleanArchitecture.src.HumblePattern.Correct
{
    public class LoginUI
    {
        private readonly TextView textView = null;
        private readonly LoginService loginService = null;

        public LoginUI()
        {
            textView = new TextView();
            loginService = new LoginService();
        }

        private void OnLoginButtonClicked()
        {
            MessageView messageView = new MessageView();
            if (loginService.OnLogin("123"))
            {
                messageView.Show("로그인 성공!");
            }
            else
            {
                messageView.Show("로그인 실패!");
            }
        }
    }
}
```

이렇게 하면, UI 코드인 `LoginUI`가 험블객체가 됩니다. 

## 프레젠터와 뷰

UI 와 로그인 로직을 분리하고, UI를 험블객체로 지정하였습니다. 하지만, 문제점이 하나 있습니다. LoginUI는 UI와 관련된 로직이 있어야 하는데, LoginService 라는 클래스를 알고 있고, 함수에서 로직을 처리하는데 사용하고 있어 캡슐화가 깨졌습니다. 

이를 해결하기 위해, `프레젠터(Presenter)`를 사용합니다. Presenter는 View와 특정로직 사이를 중재하며, View가 View 코드만 가지게 할 수 있게 합니다.

다시 말하자면, View는 View코드만 가지고 있고, Event 발생시 Event만 Presenter 객체로 넘겨주는 역할을 담당합니다. 

그러면 Presenter는 그 이벤트를 받아서 LoginService 객체를 통해 특정 로직을 수행하고, View클래스의 함수를 호출하여 화면을 띄워주게 됩니다.

바뀐 코드는 다음과 같습니다.
(DI 프레임워크 및 인터페이스 사용과 같은 과정은 생략하였습니다.)

```cs
namespace CleanArchitecture.src.HumblePattern.Presenter
{
    public class LoginUI
    {
        private readonly LoginPresenter presenter = null;

        public LoginUI()
        {
            presenter = new LoginPresenter(this);
        }

        public void OnLoginButtonClicked()
        {
            presenter.OnLogin("123");
        }

        public void ShowMessage(string _message)
        {
            MessageView view = new MessageView();
            view.Show(_message);
        }
    }
}
```

```cs
namespace CleanArchitecture.src.HumblePattern.Presenter
{
    public class LoginService
    {
        public bool OnLogin(string _password)
        {
            if (_password == "123")
                return true;
            else
                return false;
        }
    }
}
```

```cs
namespace CleanArchitecture.src.HumblePattern.Presenter
{
    public class LoginPresenter
    {
        private readonly LoginUI loginView;
        private readonly LoginService loginService;

        public LoginPresenter(LoginUI _loginView)
        {
            this.loginView = _loginView;
            this.loginService = new LoginService();
        }

        public void OnLogin(string _password)
        {
            if (loginService.OnLogin(_password))
            {
                loginView.ShowMessage("로그인 성공!");
            }
            else
            {
                loginView.ShowMessage("로그인 실패!");
            }
        }
    }
}
```

지금까지, View 코드에 집중된 코드를 `험블 객체 패턴`을 통해 View 코드와 로직이 담긴 코드로 나누고, 나눈 험블 객체(View)에서 로직을 알고있는 형태를 개선하기 위해 Presenter를 만들어 통신하게 하였습니다.

이렇게 하면, 각 모듈별로 테스트가 용이해지고 클래스별로 캡슐화가 잘 되어 유지보수에도 이점이 생기게 됩니다.