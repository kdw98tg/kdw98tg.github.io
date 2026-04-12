---
title: "[SpringBoot] 기본적인 Repository 패턴"

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

데이터베이스에 접근하여 데이터를 가져올 때, Spring Boot 프레임워크에서는 Repository 패턴을 사용하기를 권장합니다. Repository 패턴을 사용하면, 데이터베이스에 접근하는 계층을 분리할 수 있고, `CRUD`메서드를 재사용할 수 있다는 장점이 있습니다. 

이번 포스팅에서는 해당 Repository 패턴과, Jpa 프레임워크를 사용하여 기본적인 Repository를 구성해봅니다. (JpaRepository를 상속받지 않음)

## Entity 생성

우선, User라는 엔티티를 생성합니다. 이 Entity를 가지고, Jpa


