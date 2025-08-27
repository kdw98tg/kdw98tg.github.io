---
title: "[python] Python 가상환경 설정하기"

categories:
  -  Python
  
tags:
  - [python, venv, interpreter]

toc: true
toc_sticky: true

published: true

date: 2024-01-16
last_modified_at: 2024-01-16
---


## 가상환경이 필요한 이유

Python은 인터프리터 언어로 코드 실행 시 해석되며 실행됩니다. Python의 특징 중 하나는 인터프리터 버전마다 설치된 패키지를 공유한다는 점입니다. 이로 인해 동일한 Python 버전(예: Python 3.11)을 사용하는 여러 프로젝트가 있을 경우, 각 프로젝트에서 필요한 패키지의 버전이 다를 때 충돌 문제가 발생할 수 있습니다. 이러한 문제를 해결하기 위해 Python은 프로젝트별로 독립적인 환경을 제공하는 **가상환경(Virtual Environment)**이라는 개념이 도입되었습니다.**

가상환경은 프로젝트마다 Python 인터프리터와 패키지를 독립적으로 관리할 수 있는 격리된 공간을 제공합니다. 이를 통해 프로젝트 간 패키지 충돌 문제를 방지하고, 각 프로젝트가 자신만의 독립된 환경에서 실행될 수 있도록 보장합니다. 또한, 가상환경은 배포 환경과 개발 환경 간의 일관성을 유지할 수 있도록 도와줍니다. 따라서 Python으로 프로젝트를 진행할 때는 전역 환경 변수를 사용하는 대신, 가상환경을 생성하여 작업하는 것이 권장됩니다.

## VSCode 에서 파이썬 가상환경 설정

### 가상환경 생성
터미널을 열고 아래의 명령어를 입력 합니다. 저는 가상환경의 이름을 myenv 라고 지었습니다. 그리고 가상환경에 `xlwings`를 설치하는 과정을 해봅니다.

```bash
python -m venv [가상환경 이름]
```

![venv](/images/Pasted%20image%2020250116144614.png)

위 사진과 같이 명령어를 입력하면, myenv가 생성이 됩니다. 혹시나 myenv 가 생성되지 않는다면, 컴퓨터에 파이썬이 설치되어 있지 않을것입니다. 파이썬을 설치 후 실행합니다.

### 인터프리터 설정

Visual Studio Code 에서 `Ctrl + Shift + P`를 입력하여 다음과 같은 문구를 기입합니다. 그 후에, 사용할 인터프리터를 설정해줘야 합니다. 방금 만든 가상환경에 깔려있는 인터프리터를 선택합니다.


![venv](/images/Pasted%20image%2020250116145258.png)

![](/images/Pasted%20image%2020250116145341.png)

`venv` 라고 되어있는 가상환경을 선택합니다. 그리고 나서, 가상환경을 켜줘야 합니다. 켜는 방법은 다음과 같습니다. 가상환경의 경로가 Window 와 Linux/Mac 이 다르기 때문에 각 운영체제에 맞게 기입합니다.

### 가상환경 활성화

- Window OS
윈도우에서는 실행정책 때문에 실행이 안될 수 있습니다. 다음 방법을 통해서 실행 정책을 수정해야 합니다.

우선, 아래의 명령어를 쳐서 실행 정책을 확인합니다. 여기서 `Restricted`라고 되어있으면 가상환경을 킬 수 없습니다.

```bash
Get-ExecutionPolicy
```

Restricted 라고 되어 있다면 아래의 명령어를 쳐서 스크립트를 실행할 수 있도록 변경 합니다.

```bash
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

실행 권한을 변경한 후에 아래의 명령어를 기입하면 가상환경이 실행되면서 터미널 앞에 가상환경이 실행되고 있다는 표시가 나타나게 됩니다.

```bash
.\myenv\Scripts\activate
```

![activate venv](/images/Pasted%20image%2020250116150256.png)


- Mac/Linux

아래의 명령어를 쳐서 가상환경을 실행 시킵니다.

```bash
source .\myenv\bin\activate
```

### 가상환경에 패키지 설치

이제, 가상환경 세팅이 완료되었습니다. 파이썬 스크립트를 하나 만들고, pip install 을 통해서 패키지를 설치합니다. 이것이 가상환경에 잘 들어가는지 확인하면 됩니다.

우선, 지금은 `xlwings` 패키지를 설치하지 않았기 때문에, 아래의 사진처럼 경고가 뜰것입니다.

![](/images/Pasted%20image%2020250116150511.png)

이제, terminal을 열어서 pip install 을 통해서 패키지를 설치 합니다.

```bash
pip install xlwings
```

![](/images/Pasted%20image%2020250116150603.png)

설치가 완료되었습니다. 가상환경에 잘 깔려있는지 보기 위해서 가상환경-Lib 폴더로 들어가서 `xlwings` 가 있는지 확인 합니다. (또는 pip list 명령어를 통해서 확인할 수도 있습니다.)

![](/images/Pasted%20image%2020250116150756.png)

정상적으로 들어가있고, test.py 에 아까 있었던 경고도 사라지게 되었습니다.

![](/images/Pasted%20image%2020250116150822.png)

즉, 가상환경에 잘 설치되었음을 알 수 있습니다.

### 패키지 설치를 간편하게 하기

이제, 설치를 간편하기 위한 방법을 알아봅니다. 터미널을 열고 다음의 명령어를 기입합니다.

```bash
pip fress
```

![](/images/Pasted%20image%2020250116151035.png)

그러면 설치되어있는 의존성들이 나오게 됩니다. 이제 이걸 txt 파일로 만들어 관리할 것입니다. 아래 명령어를 쳐서 의존성들을 requirements.txt로 만들어 관리합니다.

```bash
pip freeze > requirements.txt
```

![](/images/Pasted%20image%2020250116151137.png)

혹시나 의존성을 다시 설치하고 싶다면, 다음의 명령어를 쳐서 간편하게 재설치 합니다.

```
pip install -r requirements.txt
```

그러면 다시 모든 패키지를 설치할 필요 없이 requirements.txt 에 있는 패키지들을 자동으로 설치하게 됩니다.

### 가상환경 끄기

가상환경을 끄고 싶다면 다음 명령어를 쳐서 끕니다.

```bash
deactivate
```

![](/images/Pasted%20image%2020250116151412.png)

해당 명령어를 쳤을 때, 앞에 초록색의 (myenv)가 사라졌다면 가상환경이 꺼졌음을 의미합니다. 그리고 가상환경을 켰을때와 마찬가지로, `interpreter`를 글로벌에 있는 것으로 선택해주면 됩니다.

