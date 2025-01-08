---
title: "[ASP.NET Core] EF Core에서 Repository Pattern을 사용하지 말아야 하는 이유"

categories:
  -  ASP .NET Core
  
tags:
  - [Web, ASP.NET Core, Unity, C#]

toc: true
toc_sticky: true

published: false

date: 2024-01-07
last_modified_at: 2024-01-07
---

ORM 에서 LazyLoading 은 성능을 망칠 수 있는 것중에 하나임

dbcontext가 제공하는 기능을 복제할 뿐이다.

왜 파사드 패턴으로 감싸냐? 그건 과한 멍청함이다.

레포지토리를 분리하는게 테스트에 용이하지 않다.

