---
title: "[Ubuntu] Ubuntu Server 디스크 만들기"

categories:
  -  Server
  
tags:
  - [Ubuntu, Server, Disk, ISO]

toc: true
toc_sticky: true

published: true

date: 2024-04-17
last_modified_at: 2024-04-17
---

우분투 서버를 설치하기 위해서는 `부팅 USB`를 만들어야 합니다.

**디스크를 만들 때, USB에 있는 모든 파일이 삭제되니, 주의하셔야 합니다.**

### 우분투 서버 ISO 다운로드

[👉👉👉 우분투 서버 ISO 다운로드 ](https://releases.ubuntu.com/focal/)

![우분투 서버 다운로드](/images/Pasted%20image%2020240417133123.png)



### 부팅 USB 만들기

Rufus 를 사용하여 부팅 USB를 만듭니다.

[👉👉👉 Rufus 사이트](https://rufus.ie/ko/)

위 사이트에 접속하여 exe를 실행 시킵니다.

![Rufus.exe](/images/Pasted%20image%2020240417134536.png)

실행시키면 다음과 같은 화면이 뜹니다.

![Rufus](/images/Pasted%20image%2020240417134610.png)

어떻게 알았는지는 모르겠지만, 제가 넣어둔 USB를 찾아놓습니다.

그다음, 선택버튼을 눌러서, 다운받은 Ubuntu-Server.ISO를 넣고, 시작버튼을 누르면 자동으로생성이 됩니다.

