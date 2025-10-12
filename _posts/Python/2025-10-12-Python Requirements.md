---
title: "[python] python requirements 파일 만들기"

categories:
  -  Python
  
tags:
  - [python]

toc: true
toc_sticky: true

published: true

date: 2025-10-12
last_modified_at: 2024-10-12
---

파이썬(python)이라는 언어는 라이브러리가 아주 잘 되어 있다는 특징이 있습니다. 하지만 라이브러리가 많은 만큼 단점으로는 의존성을 관리하는것이 어렵습니다.

그래서 python의 requirements 라는 파일을 통해서 의존성을 관리할 수 있습니다.

하나의 컴퓨터에서 작업을 한다면 그렇게까지 필요하진 않은 기능이지만, 파이썬으로 작업을 할 때는 가상환경 (virtual environment)를 활용해서 제한된 환경에서 패키지를 설치하게 됩니다. 이는 프로젝트마다 필요한 의존성이 다르기 때문입니다.

또한, github 를 통해서 프로젝트를 관리할 때에 가상환경까지 버전을 관리하는 것이 아닌, 파이썬 파일만 버전을 관리하는것이 보통이기 때문에 requirements 파일을 통해서 의존성을 관리하는것은 꼭 필요하다고 할 수 있습니다.

## requirements 파일 생성하기

현재 임시로 만든 폴더의 가상환경에서는 다음과 같은 의존성들이 있습니다.

![](/images/Pasted%20image%2020251012230135.png)

이제, requirements를 가지고 해당 numpy 라이브러리의 버전을 어떤 환경에서도 설치할 수 있도록 명시하겠습니다.

다음의 명령어를 사용하면 됩니다.

```bash
pip freeze > requirements.txt
```

해당 명령어를 사용하면 프로젝트 목록에 `requirements.txt`가 추가 됩니다.

![](/images/Pasted%20image%2020251012230323.png)

그리고 해당 파일을 열어보면 제가 설치한 라이브러의 목록이 나타나게 됩니다.

![](/images/Pasted%20image%2020251012230422.png)

여기까지하면 requirements.txt 파일을 만들고, 의존성 목록을 저장하는 것이 되었습니다.


## requirements.txt에 있는 의존성들 설치하기

requirements.txt에 저장된 의존성 정보들을 가지고 다른 환경에서 프로젝트에 필요한 의존성들을 한번에 설치하는 방법에 대해서 알아봅니다.

```bash
pip install -r requirements.txt
```

위 명령어를 치게 되면, `requirements.txt`에 있는 패키지들이 설치되는것을 알 수 있습니다.

![](/images/Pasted%20image%2020251012230704.png)

이미 설치가 되어 있기 때문에, 설치가 되어 있다는 알림이 뜨며 마무리가 되었습니다.

## 환경에 맞게 requirements 파일 설정하기

특정 OS 에서만 작동하는 OS 종속적인 라이브러리들이 있습니다. 그럴 때 사용할 수 있는 것이 `sys_platform` 입니다.

```bash
# Windows에서만 설치되는 패키지
pywin32==310; sys_platform == 'win32'
 
# Linux에서만 설치되는 패키지
some-linux-package==1.0; sys_platform == 'linux'
 
# macOS에서만 설치되는 패키지
some-mac-package==1.0; sys_platform == 'darwin'
```

이 방식을 사용하면, 특정 OS 에 맞게 의존성이 설치되어, 만약 현재의 OS를 지원하지 않는 의존성이 설치되었을때에, 오류가 나는것을 방지할 수 있습니다.

