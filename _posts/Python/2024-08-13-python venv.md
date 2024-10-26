---
title: "[python] Python 가상환경 설정하기"

categories:
  -  Python
  
tags:
  - [python, venv, interpreter]

toc: true
toc_sticky: true

published: true

date: 2024-08-15
last_modified_at: 2024-08-15
---


## 가상환경이 필요한 이유

C / C++ / C# / Java 등과 달리 python은 `interpreter`언어 입니다. 이러한 이유때문에 python은 컴파일 없이 바로 실행되며, 실행환경에 따라 결과값이 달라질 수 있습니다.

처음에 파이썬을 설치하면 윈도우 컴퓨터의 환경으로 실행이 되게 됩니다.

하지만 파이썬의 버전을 바꾸거나 한다면 기본으로 설정되어 있던 환경과 달라지게 되며, 분명 pip install 을 통하여 패키지를 설치하였어도, 해당 패키지가 없다는 에러를 맞이하곤 합니다.

그렇기때문에 프로젝트마다 `가상환경`을 만들어서 파이썬을 실행하도록 해야 합니다.

## VSCode 에서 파이썬 가상환경 설정

### 가상환경 생성
터미널을 열고 아래의 명령어를 입력 합니다.

```bash
python -m venv [폴더명]
```

![python venv](Pasted%20image%2020240813214700.png)

저는 폴더명을 `venv`로 설정하였습니다.

### 인터프리터 설정

Visual Studio Code 에서 `Ctrl + Shift + P`를 입력하여 다음과 같은 문구를 기입합니다.

`select interpreter`

![](Pasted%20image%2020240813215049.png)

venv 라고 되어있는 인터프리터를 선택합니다.

이렇게 하고 나서, 아래의 터미널을 확인해야 합니다.

새로운 터미널이 열리며, 이제부터 패키지를 설치할 때 마다 venv에 설치가 되며, 정상적으로 패키지를 사용할 수 있게 됩니다.