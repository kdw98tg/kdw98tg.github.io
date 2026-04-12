---
title: "[Winform] Winform MVP 패턴 구축"

categories:
  -  Winform
  
tags:
  - [Winform, C#, MVP]

toc: true
toc_sticky: true

published: true

date: 2025-05-02
last_modified_at: 2025-05-02
---

이번에 Winform 프레임워크를 사용하여 지휘봉을 움직이면 그에따라 음악의 볼륨, 템포를 조절하는 외주를 하였습니다. 외주를 하면서 프로그램적으로 느낀것은 크게 2가지가 있습니다.

첫번째는, 고객의 요구사항이 자주 바뀐다 입니다. 이점은 인터페이스를 사용하여 해결할 수 있었습니다.
두번째는, 한 폼에 기능이 너무 많아지니까, 관리하기가 힘들다 였습니다. 어떤 수정사항이 생겼을 때, 어떤 기능을 담당하는 모듈만 보면 되게끔 바꾸는게 좋아보였습니다. 그렇게 관심사를 분리하여 MVP 패턴을 적용하여 문제를 해결했습니다.

이번 포스팅에서는 두번째 문제를 해결하며 사용하였던 MVP 패턴을 소개합니다.

## 아키텍쳐 패턴이란?

MVP 패턴을 소개하기 위해서는 아키텍쳐 패턴이란것을 먼저 다뤄야 합니다. 아키텍쳐 패턴은, 소프트웨어 설계에서 반복적으로 발생하는 문제에 대한 일반적인 해결책을 제공하는 템플릿과 같은 것입니다. 특정 프레임워크의 특징이나 특성을 이해하고, 특정 프레임워크에 적합한 구조를 찾는 과정이라고 볼 수 있습니다.

아키텍쳐 패턴을 사용하면, 확장성, 유지보수성을 챙길 수 있다는 장점이 있지만, 단점으로는 코드를 한번에 알아보기 힘들 수 있다는 단점이 있습니다.

아키텍쳐 패턴중에 가장 유명한 것은 MVC, MVP, MVVM 등이 있습니다. 

## MVP 패턴

아키텍쳐 패턴 중에서 MVP 패턴은 Model - View - Presenter 의 약자이며, Presenter가 View 와 Model 을 중재하며 로직을 작성하는것이 중점입니다. MVP 패턴 없이 코드를 작성하게 되면, 보통 View 코드에 코드가 집중되는 현상이 벌어집니다. 하지만 MVP 패턴을 사용하면 관심사를 분리하여 View는 View에 관한 코드만 작성하게 할 수 있고, Model 은 비지니스 로직에 관한 코드만 작성할 수 있게 합니다.

MVP 패턴의 구조는 다음과 같습니다.

![](/images/Pasted%20image%2020250502125234.png)

Presenter는 Model과 View를 알고 있고, Model 과 View는 서로를 모르는 구조입니다. 

## Winform에 적용

MVP패턴을 Winform에 적용한 예제 입니다. 해당 예제 코드는 아래의 깃헙 주소에서 코드를 받을 수 있습니다.

https://github.com/kdw98tg/Winform_MVP_Structure

해당 예제는 User 정보를 가져오고, User를 저장하는 예제입니다. (데이터베이스는 간단하게 휘발성 메모리 데이터베이스를 사용했습니다.)

또한, MVP 패턴을 구성할 때, 이론적으로는 View가 콜백으로 Presenter에게 이벤트를 알려주게 되어있지만, 콜백을 사용하면 코드 가독성이 떨어질 수 있다고 생각하여 Interface를 중간에 두고, Presenter의 인터페이스를 참조하는 식으로 구성하였습니다. 

![](/images/Pasted%20image%2020250502130110.png)

### View 만들기

(만들어진 코드를 예시로 듭니다.)

우선, View 인터페이스를 만듭니다. 해당 인터페이스는 View라면 공통적으로 해야할 기능들을 정의한 인터페이스 입니다. 각 세부 View들은 해당 인터페이스를 상속받아서 구현을 하게 됩니다. 우선, MessageBox를 띄우는 기능들만을 정의해 놓았습니다.

```cs
namespace MVP_Structure.View
{
    public interface IView
    {
        public void ShowMessageBox(string? _text);
        public void ShowMessageBox(string? _text, string? _caption);
        public void ShowMessageBox(string? _text, string? _caption, MessageBoxButtons _buttons, MessageBoxIcon _icon);
    }
}

```

다음은, MainForm의 View를 만듭니다. MainForm의 View도 마찬가지로, 인터페이스를 작성하고, MainForm에 상속시켜 줍니다.

```cs
using MVP_Structure.Model;

namespace MVP_Structure.View.MainFormView
{
    public interface IMainFormView : IView
    {
        public void AddUserToListView(User _userList);
    }
}
```

```cs
using MVP_Structure.Model;
using MVP_Structure.Presenter.MainForm;
using MVP_Structure.View;
using MVP_Structure.View.MainFormView;

namespace MVP_Structure
{
    public partial class MainForm : BaseForm, IMainFormView
    {
        private IMainFormPresenter presenter = null;

        public MainForm()
        {
            InitializeComponent();
        }
        
        public void Init(IMainFormPresenter _presenter)
        {
            presenter = _presenter;
        }
    }
        //....
}
```

코드를 그대로 복붙해서 IMainFormPresenter 코드가 있지만, 새로 만드시는 경우에는 해당 인터페이스가 없기에, 새로 만들어 줍니다.

### Presenter 만들기

View와 마찬가지로, Presenter의 인터페이스를 만들고, 구현체를 만들도록 합니다.

```cs
using MVP_Structure.Model;

namespace MVP_Structure.Presenter.MainForm
{
    public interface IMainFormPresenter 
    {
        public void InitUserListView();
        public void InsertUser(User _user);
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

        public MainFormPresenter(IMainFormView presenter)
        {
            mainView = presenter;
            userService = new UserService(new UserRepository());
        }
    }
}
```

지금 Presenter에 보면, Service라는 객체가 있는데, 해당 Service는 이 프로젝트의 도메인 로직을 구현해놓은 클래스 입니다. 

userService에서 모델들을 관리하고, 비지니스 로직을 정의하기 때문에, 해당 클래스가 Model이라고 생각해주시면 됩니다.

### Model 만들기

이제 모델을 만듭니다. 모델은 View나 Presenter처럼 상호 의존성을 막기위해 interface를 만들지 않아도 됩니다. 또한, 모델이 바뀌는경우는 크게 없기 때문에 굳이 interface로 추상화 하지 않습니다.

```cs
namespace MVP_Structure.Model
{
    public class User
    {
        public int UserId { get; set; }
        public string UserName { get; set; }
        public int UserAge { get; set; }
    }
}
```

다음은 User를 데이터베이스에서 조작할 Repository를 만듭니다. Repository는 데이터베이스가 바뀐다던지, 데이터베이스를 조작할 라이브러리가 바뀐다던지의 경우가 있기 때문에, interface로 감싸서 구현합니다.

```cs
using MVP_Structure.Model;

namespace MVP_Structure.Repository
{
    public interface IUserRepository
    {
        public List<User> GetAllUser();
        public void InsertUser(User _newUser);
        public void DeleteUser(User _user);
    }
}
```

```cs
using MVP_Structure.Model;

namespace MVP_Structure.Repository
{
    public class UserRepository : IUserRepository
    {
        private List<User> userList = null;

        public UserRepository()
        {
            userList = new List<User>();
            userList.Add(new User() { UserId = 1, UserName = "홍길동", UserAge = 12 });
        }

        public List<User> GetAllUser()
        {
            return userList;
        }

        public void InsertUser(User _newUser)
        {
            userList.Add(_newUser);
        }

        public void DeleteUser(User _user)
        {
            userList.Remove(_user);
        }
    }
}
```

그리고 해당 Repository를 사용하여 도메인 로직을 구현할 Service로 갑니다.

```cs
using MVP_Structure.Exceptions;
using MVP_Structure.Model;
using MVP_Structure.Repository;

namespace MVP_Structure.Service
{
    public class UserService
    {
        private IUserRepository userRepository = null;

        public UserService(IUserRepository _userRepository)
        {
            this.userRepository = _userRepository;
        }

        public List<User> GetAllUsers()
        {
            return userRepository.GetAllUser();
        }

        public void InsertUser(User _newUser)
        {
            ValidateUser(_newUser);

            userRepository.InsertUser(_newUser);
        }

        public void ValidateUser(User _newUser)
        {
            if (_newUser.UserAge <= 0)
            {
                throw new UserAgeException("나이는 음수일 수 없음");
            }
            if (string.IsNullOrEmpty(_newUser.UserName))
            {
                throw new UserNameException("이름을 넣어주세요.");
            }
        }

        public void DeleteUser(User _user)
        {
            userRepository.DeleteUser(_user);
        }
    }
}
```

이렇게 하면, MVP의 기본적인 구성이 끝나게 됩니다.

### Program.cs에서 의존성 주입

마지막으로, DI할 객체들을 Program.cs 에서 넣어주기만 하면 됩니다.

```cs
using MVP_Structure.Presenter.MainForm;
namespace MVP_Structure
{
    internal static class Program
    {
        /// <summary>
        ///  The main entry point for the application.
        /// </summary>
        [STAThread]
        static void Main()
        {
            // To customize application configuration such as set high DPI settings or default font,
            // see https://aka.ms/applicationconfiguration.
            ApplicationConfiguration.Initialize();

            MainForm view = new MainForm();
            IMainFormPresenter presenter = new MainFormPresenter(view);
            view.Init(presenter);
            Application.Run(view);
        }
    }
}
```


## 보강해야할 점

MVP 패턴에서 치명적인 단점은 프로그램의 사이즈가 늘어날수록, Presenter가 비대해진다는 것입니다. 결국 그러면 처음에 직면했던, 코드가 너무 길어져서 관리하기가 힘들다는 단점이 부각되게 됩니다. 해당 문제를 해결하기위해, 다중 View를 사용해서 구성해봤지만, 쓸데없이 코드가 더 복잡해지고, 인터페이스가 너무 많아져서, 디버깅이 힘들다는 단점이 있었습니다.

또한, 지금 제가 만든 데모에는 DI프레임워크가 도입되어 있지 않습니다. DI프레임워크를 도입한다면 의존성 주입을 더 간편하게 하여 유지보수에 더 큰 도움을 줄 수 있을것 같습니다.

