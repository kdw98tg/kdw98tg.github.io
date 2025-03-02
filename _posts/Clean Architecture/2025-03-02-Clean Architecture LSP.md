---
title: "[Clean Architecture] Clean Architecture LSP(Liskov Substitution Principle 라스코프 치환 원칙)"

categories:
  -  Clean Architecture
  
tags:
  - [Clean Architecture]

toc: true
toc_sticky: true

published: true

date: 2025-03-02
last_modified_at: 2025-03-02
---

## 라스코프 치환 원칙 (LSP)

1988년 바바라 라스코프는 하위 타입을 아래와 같이 정의하였습니다.

> 여기에서 필요한 것은 다음과 같은 치환 원칙이다. S타입의 객체 o1 각각에 대응하는 T 타입 객체 o2가 있고, T 타입을 이용해서 정의한 모든 프로그램 P에서 o2의 자리에 o1을 치환하더라도 P의 행위가 변하지 않는다면 S는 T의 하위 타입이다.

즉, 간단하게 말하면 자식타입은 언제나 부모타입을 교체할 수 있어야 한다는 것입니다.

리스코프 치환 원칙은 이름만 봐서는 어떤 원칙인지 상상하기 어렵습니다. 그래서 대표적인 라스코프 치환 원칙을 위반한 예제를 살펴봅시다. 

## 라스코프 치환 원칙 위반 예제 (정사각형/직사각형 문제)

우선, 정사각형과 직사각형의 특성에 대해 생각해 봅시다. 직사각형은 높이와 너비가 서로 다를 수 있지만, 정사각형은 높이와 너비가 서로 같은 사각형을 말합니다.

코드로 간단한 예시를 만들면 다음과 같습니다.
