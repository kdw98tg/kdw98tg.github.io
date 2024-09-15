---
title: "[Unity] Unity 빌드시 멈추는 현상 해결"

categories:
  -  Unity
  
tags:
  - [Unity, Error, Build]

toc: true
toc_sticky: true

published: true

date: 2024-09-15
last_modified_at: 2024-09-15
---

빌드 테스트를 하다가, 빌드 세팅을 완료하고 빌드가 시작됐을 때, 갑자기 유니티가 뻗어버리는 현상이 있었습니다.

유니티가 뻗어버리니까 Error 로그도 확인할 수 없어서, 꽤나 많은 시간을 버그 해결하는데 투자한 것 같습니다.

이유는 너무나 간단했습니다.

![](/images/Pasted%20image%2020240915121108.png)

빌드하는 타겟기기의 CPU 설계 방식에 따라 빌드가 되는데, 제 타겟 기기의 아키텍쳐에 해당하는 설계 방식이 체크되어있지 않아 문제가 발생하였습니다.

ARMv7을 체크하고 빌드를 하니 멈춤현상 없이 빌드가 수행되었습니다.

