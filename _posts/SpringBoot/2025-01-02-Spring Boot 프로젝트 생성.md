---
title: "[SpringBoot] Spring Boot 프로젝트 생성"

categories:
  - SpringBoot
  
tags:
  - [Java, Spring Boot]

toc: true
toc_sticky: true

published: false

date: 2025-06-06
last_modified_at: 2025-06-06
---

Spring Boot 를 시작하려면, 여러 의존성들을 설치해줘야 합니다.

Java 17, Gradle, SpringBoot 등등이 대표적인 예 입니다.

그것을 간편하게 해주는 사이트가 있습니다. 

[https://start.spring.io/]

해당 사이트에 들어가서 필요한 의존성들을 추가한 뒤, 아래의 Generate를 선택해주면 기본적인 스프링 프로젝트가 다운로드 되게 됩니다.

![](/images/Pasted%20image%2020250606201119.png)

기본적으로 `Spring Web`, `Lombok`, `MySqlDriver`을 설치하고, 추가로 템플릿 엔진이 필요하다면 `Thymleaf` 패키지를 설치해 줍니다.

Zip파일을 다운로드 받았다면, `build.gradle`을 `IntellJ`로 열어서 프로젝트를 열면 됩니다.

