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

하지만, 이런식으로 싱글턴 패턴을 구상하게 되면, 멀티스레드 문제가 생길 수 있습니다.

`if(uniqueInstance == null)` 이부분이 실행되고, 이제 instance 를 생성하게 될때, 다른 스레드에서 널 체크를 하게 되면, 객체가 2개가 생겨버리는 것입니다.

다음 사진과 같은 경우 입니다.

![싱글턴 멀티스레드 문제](/images/Pasted%20image%2020240304233316.png)

## 예제 - getInstance() 동기화

싱글턴의 멀티 스레드 문제를 해결하기 위한 방법으로 `getInstance()` 메서드를 동기화 하는 방법이 있습니다.

구현은 다음과 같이 합니다.
🗅 **<span style="color: #c03a92">class ThreadSafeSingleton</span>**
```java
public class ThreadSafeSingleton {  
    private ThreadSafeSingleton (){};  
    private static ThreadSafeSingleton Instance;  
      
    //synchronized 키워드를 넣어서 멀티스레드 문제를 해결  
    //synchronized 키워드를 넣으면 한 스레드가 메소드 사용을 끝내기 전까지 다른 스레드는 기다림  
    //하지만 동기화에서의 속도저하가 일어날 수 있음  
    //성능이 100배정도 저하될 수 있지만, 속도가 문제가 아니라면 걍 사용하면 됨  
    public static synchronized ThreadSafeSingleton getInstance(){  
        if(Instance == null){  
            Instance = new ThreadSafeSingleton();  
        }  
        return Instance;  
    }  
}
```

`syncronized`키워드를 넣으면 한 스레드가 메소드 사용을 끝내기 전까지 다른 스레드는 기다리게 됩니다.

하지만, `기다린다` 라는 점 떄문에 동기화로 멀티스레드 문제를 해결할 때 다음과 같은 문제가 발생할 수 있습니다.

### getInstance() 동기화의 문제점
- 동기화에서의 속도저하가 일어날 수 있음
- 속도저하는 약 100배 정도가 됨.

이를 해결 하는 방법에는 2가지가 있습니다.


## 예제 - 더 효율적으로 멀티 스레드 문제 해결

### Instance 미리 생성

첫번째 방법은 정말 간단합니다.

`null`체크를 하지 않고, 미리 인스턴스를 만들어 놓는 것입니다.

🗅 **<span style="color: #c03a92">class StaticSingleton</span>**
```java
public class StaticSingleton {  
  
    //인스턴스가 필요할 때는 생성하지말고 처음부터 만드는게 좋음  
    //synchroized 를 안쓰고 구현하려면 이렇게 하면 됨  
    private static StaticSingleton Instance = new StaticSingleton();  
  
    private StaticSingleton(){}  
  
    public static StaticSingleton getInstance(){return Instance;}  
  
}
```

하지만, 여기에도 문제가 있습니다.

미리 객체를 생성해서 속도를 향상시킨 것은 좋지만, 말그대로 언제 사용할지 모를 인스턴스를 만들어 놓는 것은 메모리 낭비일 수 있습니다.

그래서, 다른 해결법이 등장합니다.

## 예제 - DCL 을 사용해서 동기화 되는 부분 줄이기

`DCL(Double-Checked Locking)`을 사용하면 인스턴스가 생성되어 있는지 확인한 다음 생성되어 있지 않았을 때만 동기화할 수 있습니다.

이러면, 처음에만 동기화 하고 나중에는 동기화하지 않아도 됩니다.

🗅 **<span style="color: #c03a92">class DCLSingleton</span>**
```java
public class DCLSingleton {  
  
    //volatile 키워드를 사용하면 멀티스레딩을 쓰더라도,  
    // uniqueInstance 변수가 Singleton 인스턴스로 초기화되는 과정이 올바르게 진행됨  
    //이런식으로 구현하면, getInstance()메소드를 사용할 때 발생하는 속도를 극적으로 줄일 수 있음  
    //dcl 은 자바 1.4 버전 이전에는 사용할 수 없음  
    private volatile static DCLSingleton uniqueInstance;  
  
    private DCLSingleton (){}  
  
    public static DCLSingleton getInstance(){  
        if(uniqueInstance == null){  
            //인스턴스가 있는지 확인하고, 없으면 동기화된 블록으로 들어감  
            //이렇게하면 처음에만 동기화 됨  
            synchronized (DCLSingleton.class){  
                if(uniqueInstance == null){  
                    uniqueInstance = new DCLSingleton();  
                }  
            }  
        }  
        return uniqueInstance;  
    }  
}
```

`volatile`키워드를 사용하면 멀티스레딩을 하더라도, uniqueInstance 변수가 Singleton 인스턴스로 초기화되는 과정이 올바르게 진행 됩니다.

이런식으로 구현하면, `getInstance()`메소드를 사용할 때 발생하는 속도 저하를 극적으로 줄일 수 있습니다.

하지만, 여기에도 단점이 존재합니다.

`java 5` 보다 낮은 버전의 JVM을 써야 한다면, 다른 방식으로 싱글턴을 구현해야 합니다.


## 예제 - Enum을 활용한 싱글톤

지금까지 논의한 모든 문제를 `enum`으로 싱글턴을 생성해서 해결 할 수 있습니다.

```java
public enum Singleton {  
    UNIQUE_INSTANCE;  
  
    // other useful fields here  
  
    // other useful methods here    
    public String getDescription() {  
        return "I'm a thread safe Singleton!";  
    }  
         
}
```

정말 간단하게 구현이 됩니다.

하지만 해당 문법은 `java`에서만 동작합니다.

그래서 각 언어에서 싱글톤을 안전하게 쓰기 위한 방법들을 찾아보는 것이 중요할 것 같습니다.

그래도 역시, **싱글턴은 객체지향의 철학에도 어긋나고, 단단한 결합을 만들어내어 테스트에도 불리한 점이 있기에 자주 사용하지 않는 것이 좋습니다.**