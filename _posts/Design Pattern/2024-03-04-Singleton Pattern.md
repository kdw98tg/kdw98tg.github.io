---
title: "[Design Pattern] 싱글턴 패턴 (Singleton Pattern)"

categories:
  -  Categories
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2024-03-04
last_modified_at: 2024-03-04
---

## 싱글톤 패턴이란?

>싱글톤 패턴이란 단 하나의 유일한 객체를 만들기 위한 코드 패턴 입니다.
>
>메모리 절약을 위해, 인스턴스가 필요할 때 똑같은 인스턴스를 새로 만들지 않고, 기존의 인스턴스를 가져와 활용합니다.
>
>또한, 하나의 값만을 참조해야 할 때, 싱글톤 패턴을 사용합니다.


### 싱글톤 패턴의 장점
- 인스턴스가단 하나이기 때문에 인스턴스를 생성할 때 드는 비용이 줄어듦
- 싱글톤으로 만들어진 클래스의 인스턴스는 전역 인스턴스 이기 때문에 다른 클래스의 인스턴스들이 데이터를 공유할 수 있음

### 싱글톤 패턴의 단점
- 싱글톤 인스턴스를 여러곳에서 많이 공유할 경우, 다른 클래스의 인스턴스 간 의존성이 높아짐
- 의존성이 높아지면 테스트하기 힘듦
- 멀티스레드 환경에서 오류가 날 수 있음(동기화로 해결할 수 있긴 함)


싱글톤 패턴은 고전적인 형태가 있고, 그 형태의 문제점들을 개선해 나가며 새롭게 구성된 형태들이 있습니다.

우선, 고전적인 패턴부터 하나씩 만들어 봅니다.


## 예제 - 고전적인 싱글턴 패턴 구현

고전적인 싱글턴패턴은 단순합니다.

객체의 인스턴스 변수를 하나 지정하고, 그 변수를 getter 메소드로 호출하면서, 객체의 유일성을 보장하는 방식입니다.

그리고, 다른 외부에서 해당 클래스를 생성하면 안되므로, 생성자를 `private`으로 만들어 줍니다.

🗅 **<span style="color: #c03a92">class Main</span>**
```java
public class ClassicSingleton {  
    private static ClassicSingleton uniqueInstance;  
  
    private ClassicSingleton(){}  
  
    public static ClassicSingleton getInstance(){  
        if(uniqueInstance == null){  
            uniqueInstance = new ClassicSingleton();  
        }  
        return uniqueInstance;  
    }  
  
    // other useful methods here  
    public String getDescription() {  
        return "싱글톤 객체 입니다.";  
    }  
}
```

