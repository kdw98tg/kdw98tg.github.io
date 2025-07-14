---
title: "[Window] Visual Studio Code Profile 넣기"

categories:
  - Etc

tags:
  - Visual Studio Code
  - Window

toc: true
toc_sticky: true

published: true

date: 2025-07-14
last_modified_at: 2025-07-14
---


Visual Studio Code는 강력한 코드 에디터입니다.  
하지만 단점 중 하나는 다양한 확장 기능(Extension)을 통해 개발 환경을 구성해야 하기 때문에, 여러 컴퓨터에서 동일한 환경을 유지하기 어렵다는 점입니다.

이를 해결하기 위해 **프로필(Profile)** 기능을 활용하면 됩니다.  
프로필을 파일로 저장하고, 이 파일을 가져와 활성화(Activate)하면 어떤 컴퓨터에서도 동일한 개발 환경을 쉽게 복원할 수 있습니다.

우선, Visual Studio Code를 켜고, File - Share - Exprot Profile()... 을 선택합니다.

![](/images/Pasted%20image%2020250714172703.png)

Export  Profile을 선택해서 들어가게 되면, 아래와 같은 화면이 나오게 됩니다. (처음 들어가면 Default 프로파일  하나만 존재하게 됩니다.)
![](/images/Pasted%20image%2020250714172749.png)

여기서, New Profile의 오른쪽에 있는 화살표 버튼을 클릭하여, Import Profile을 선택해 줍니다.

![](/images/Pasted%20image%2020250714172917.png)

Import Profile을 선택하면, Select File 이라는 항목이 나오게 됩니다. 저것을 클릭하여 파일 탐색기를 열어줍니다.

![](/images/Pasted%20image%2020250714173013.png)

파일 탐색기를 열면, profile 파일을 찾아서 `열기`버튼을 눌러줍니다.

![](/images/Pasted%20image%2020250714173112.png)

`열기` 버튼을 누르고 난 뒤에, 새로 들어온 Profile 을 선택하여 오른쪽의 `체크표시`를 눌러주게 되면, Active 라는 글자와 함께, Profile이 설정된것을 볼 수 있습니다.