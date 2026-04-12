![](Pasted%20image%2020250605170830.png)

![](Pasted%20image%2020250605170903.png)![](Pasted%20image%2020250605171003.png)![](Pasted%20image%2020250605171030.png)![](Pasted%20image%2020250605171116.png)![](Pasted%20image%2020250605171156.png)![](Pasted%20image%2020250605171211.png)![](Pasted%20image%2020250605171300.png)![](Pasted%20image%2020250605171415.png)![](Pasted%20image%2020250605172137.png)![](Pasted%20image%2020250605172205.png)![](Pasted%20image%2020250605172235.png)

**참고: 강제로 scope를 email과 profile로 등록한 이유는?**  
scope의 기본값은 `openid`, `email`, `profile`이다. 하지만 `openid`라는 scope가 있으면 OpenId Provider로 인식한다. 그렇게 되면 **OpenId Provider인 서비스(구글)**와 **그렇지 않은 서비스(네이버, 카카오 등)**로 나눠서 **각각 `OAuth2Service`를 만들어야 한다.** 따라서 하나의 `OAuth2Service`를 사용하기 위해 일부러 `openid` scope를 빼고 등록한다.

provider, openid
https://choihjhj.tistory.com/entry/springboot-%EA%B5%AC%EA%B8%80-%EB%A1%9C%EA%B7%B8%EC%9D%B8-api-%EC%97%B0%EB%8F%99#google_vignette

https://cn-c.tistory.com/124