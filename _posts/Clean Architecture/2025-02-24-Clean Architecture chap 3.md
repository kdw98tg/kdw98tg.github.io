---
title: "[Clean Architecture] Clean Architecture chap 3 정리"

categories:
  -  Clean Architecture
  
tags:
  - [Clean Architecture]

toc: true
toc_sticky: true

published: true

date: 2025-02-24
last_modified_at: 2025-02-24
---

좋은 소프트웨어 시스템은 깔끔한 코드로부터 시작합니다. 좋은 벽돌을 사용하지 않는다면, 좋은 빌딩이 나올 수 없기 때문입니다. 반대로 좋은 벽돌을 사용하더라도, 건물의 설계가 엉망이라면 좋은 건물이 탄생할 수 없습니다. 그래서 좋은 벽돌로 좋은 설계를 정의하는 원칙이 필요한데, 그것을 SOLID 원칙이라고 합니다.

SOLID 원칙은 함수와 데이터 구조를 클래스로 배치하는 방법, 그리고 이들 클래스를 서로 결합하는 방법을 설명해 줍니다. SOLID 원칙의 목적은 다음과 같습니다.

- 변경에 유연하다.
- 이해하기 쉽다.
- 많은 소프트웨어 시스템에 사용될 수 있는 컴포넌트의 기반이 된다.

이들 원칙들은 다음과 같습니다.

SRP(Single Responsibility Principle) 단일책임원칙
- 소프트웨어 모듈의 변경의 이유는 하나여야 한다. 단 하나여만 한다.

OCP(Open-Closed Principle) 개방폐쇄원칙
- 소프트웨어는 확장에는 열려있어야 하고 변경에는 닫혀있어야 한다.

LSP(Liskov Substitution Principle) 리스코프 치환원칙
- 하위 타입은 상위타입을 언제든지 대체할 수 있어야 한다.

ISP(Interface Segregation Principle) 인터페이스 분리의 원칙
- 인터페이스를 잘게 분리해서 사용자는 사용하지 않은 것에 의존하지 않아야 한다.

DIP(Dependency Inversion Principle) 의존성 역전의 원칙
- 고수준 정책을 구현하는 코드는 저수준 세부사항을 구현하는 코드에 절대로 의존해서는 안된다. 대신 세부사항이 정책에 의존해야 한다.

지금까지 Soild 원칙이 중요하고, 지향해야할 방향이라는것을 알고, 그것을 지키려고 했습니다만, 이것들을 포스팅 한적이 없어 이번 기회에 하나씩 포스팅 해볼까 합니다.

다음포스팅부터 하나씩 다뤄보겠습니다.