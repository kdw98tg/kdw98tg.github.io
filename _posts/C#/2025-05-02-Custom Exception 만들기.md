---
title:  "[C#] C# Custom Exception 만들기" 

categories:
  -  C Sharp
tags:
  - [Programming, Language, Exception]

toc: true
toc_sticky: true

published: true

date: 2025-05-02
last_modified_at: 2025-05-02
---

예외를 처리하는것은 프로그램에서 아주 중요한 사항입니다. 라이브러리를 만들어서 배포할때, 또는 디버깅을 위해서 등등 여러곳에서 사용하게 됩니다.

C#에서는 Exception 이라는 클래스를 상속받은 여러가지 예외 클래스들이 있습니다. 프로그램을 시작했다면 한번은 접해본 `ObjectNullReferenceException`, `ArgumentNullException`, `ArgumentException` 등이 그 예 입니다.

하지만, 해당 예외만 가지고 도메인에서 발생하는 예외들을 처리하기에는 힘듭니다. 따라서, 커스텀으로 예외를 만들어서 사용해야 합니다.

C#에서 예외를 만드는 방법은 다음과 같습니다.

## Excpetion 만들기

우선, 클래스를 만들고, 클래스의 이름을 원하는 Exception 이름으로 정의합니다. User의 이름이 잘못되었음을 나타내는 예외를 만들도록 하겠습니다.


```cs
    class UserNameInvalidException : Exception
    {
    
    }
```

클래스를 만든 후, 생성자를 총 3개 필수적으로 생성해줘야 합니다. (기본 생성자를 제외하면 2개)

```cs
class UserNameInvalidException : Exception
{
	public UserNameInvalidException()
	{

	}

	public UserNameInvalidException(string _message) : base(_message)
	{

	}

	public UserNameInvalidException(string _message, Exception _inner) : base(_message, _inner)
	{

	}
}
```

이렇게 하면 사용하는 곳에서 다음과 같이 쓸 수 있습니다.

```cs
public class UserService
{
	public void ValidateUser(User _newUser)
	{
		if (_newUser.UserAge <= 0)
		{
			throw new UserAgeException("나이는 음수일 수 없음");
		}
		if (string.IsNullOrEmpty(_newUser.UserName))
		{
			throw new UserNameInvalidException("이름을 넣어주세요.");
		}
	}
}
```

```cs
using MVP_Structure.Exceptions;
using MVP_Structure.Model;
using MVP_Structure.Repository;
using MVP_Structure.Service;
using MVP_Structure.View.MainFormView;

namespace MVP_Structure.Presenter.MainForm
{
    public class MainFormPresenter : IMainFormPresenter
    {
        private IMainFormView mainView = null;

        private UserService userService = null;

        public MainFormPresenter(IMainFormView _view)
        {
            mainView = view;
            userService = new UserService(new UserRepository());
        }
        
        public void InsertUser(User _user)
        {
            try
            {
                userService.InsertUser(_user);
                mainView.AddUserToListView(_user);
            }
            catch (UserNameInvalidException _userNameException)
            {
                MessageBox.Show("이름 잘못됨" + _userNameException.Message);
            }
        }
    }
}
```

이렇게 하여, 특정 오류마다 Handling을 할 수 있게 된다는것이 가장 큰 장점인것 같습니다.

