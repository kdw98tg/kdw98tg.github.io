---
layout: post

title: "[Design Pattern] 옵저버 패턴 (Observer Pattern)"

categories:
  -  Design Pattern
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

date: 2023-05-13
last_modified_at: 2024-03-02
---

## 옵저버 패턴이란?

>옵저버 패턴(Observer Pattern)은 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체에게 연락이 가고, 자동으로 내용이 갱신되는 방식으로 일대다(one-to-many) 의존성을 정의합니다.

### 옵저버 패턴이 사용되는 곳
- 앱이 한정된 시간, 특정한 케이스에만 다른 객체를 관찰해야 하는 경우
- 대상 객체의 상태가 변경될 때마다 다른 객체의 동작을 트리거할 경우
- 한 객체의 상태가 변경되면 다른 객체도 변경해야 할 때. 변경하면서 어떤 객체가 변경되어야 하는지 몰라도 될 때

### 옵저버패턴의 장점
- Subject의 상태 변경을 주기적으로 조회하지 않고, 정보 변경 시, 자동으로 감지 가능
- 발행자의 코드를 변경하지 않고, 새 구독자를 등록할 수 있어 `개방 폐쇠 원직(OCP)`를 준수
- 런타임에서 발행자와 구독 알림 관계를 맺을 수 있음
- `정보를 변경하는 객체(Subject)`와 `변경을 감지하는 객체(Observer)`의 관계를 느슨하게 유지할 수 있음`(느슨한 결합)`

### 옵저버패턴의 단점
- 구독자는 알림 순서를 제어 불가능. 무작위의 순서로 알림을 받음 (구현가능 하지만, 추천 x)
- 옵저버 패턴 남발시, 구조와 동작을 알아보기 힘듦 -> 코드 복잡도 증가
- 다수의 옵저버 객체를 등록 이후, 해지하지 않는다면 메모리 누수 발생

### 사용된 디자인 원칙
> 느슨한 결합 (Loose Coupling)
> 
> 상호작용하는 객체 사이에는 가능하면 느슨한 결합을 사용해야 합니다.


옵저버 패턴에는 `Subject 객체`가 정보를 등록된 `Observer`에게 **push** 해주는 방식과 필요한 정보를 `Observer 객체`가 가져오는 **pull** 방식 으로 나뉩니다.

## 예제 - push 방식

우선 기상 정보를 Observer들에게 뿌려주는 상황을 가정해 봅시다.

정보를 전달하는 `WeatherData` 라는 클래스가 있습니다. 이 클래스는 정보가 변경될 때 마다 각 디스플레이에 정보를 전달합니다.

각 디스플레이는 Display와 Observer의 역할을 둘다 하게 됩니다. 즉, 정보를 전달받는 `Observer` 인터페이스와 화면을 표시하는 `Display`인터페이스를 동시에 상속받게 됩니다.

처음 구현할 방식은 push 방식 입니다.

`Subject`에서 정보가 바뀌면, 등록된 `Observer` 객체에게 연락을 싹 다 돌리는 방식 입니다.

우선, `Observer` 객체와 `DisplayElements`, `Subject`를 정의할 interface를 만들도록 합니다.

🗅 **interface Observer**
```java
public interface Observer {  
  
    //옵저버들이 가질 update 메소드  
    //온도, 습도, 기압 정보를 update   
     public void update(float _temp, float _humidity, float _pressure);
}
```

🗅 **interface DisplayElements**
```java
public interface DisplayElement {  
  
    //표시장치에 있는 display 함수  
    public void display();  
}
```

🗅 **interface Subject**
```java
public interface Subject {  
  
    //옵저버 등록  
    public void registerObserver(Observer _observer);  
  
    //옵저버 제거  
    public void removeObserver(Observer _observer);  
  
    //옵저버에게 정보가 변경됨을 알리기  
    public void notifyObservers();  
}
```

### 각 Observser, Subject 구현

이제 인터페이스를 만들었으니, 해당 인터페이스를 상속받아 구현할 구상 클래스를 만듭니다.

다음과 같이 구성을 해봅니다.

- Subject : `WeatherData`
- Observer : `ForecastDisplay`, `CurrentConditionsDisplay`

#### Subject 클래스 구현
먼저, Subject 클래스부터 구현을 합니다.

Subject 클래스에는 옵저버를 등록/해지 할 수 있는 함수와, 옵저버에게 연락을 돌리는 함수로 구성되어 있습니다.

🗅 **class WeatherData**
```java
public class WeatherData implements Subject{  
  
    private List<Observer> observerList;  
    private float temperature;  
    private float humidity;  
    private float pressure;  
  
    public WeatherData(){  
        observerList = new ArrayList<Observer>();  
    }  
  
  
    @Override  
    public void registerObserver(Observer _observer) {  
        observerList.add(_observer);  
    }  
  
    @Override  
    public void removeObserver(Observer _observer) {  
        observerList.remove(_observer);  
    }  
  
    @Override  
    public void notifyObservers() {  
        for(Observer observer : observerList){  
            observer.update(temperature,humidity,pressure);  
        }  
    }  
  
    public void measurementsChanged(){  
        notifyObservers();  
    }  
  
    public void setMeasurements(float _temperature, float _humidity, float _pressure){  
        this.temperature = _temperature;  
        this.humidity = _humidity;  
        this.pressure = _pressure;  
        measurementsChanged();  
    }
```

#### Observer 클래스 구현

Observer 클래스는 Observer, DisplayElements를 상속받아 옵저버의 역할과 Display 의 역할을 수행 합니다.

그리고 Subject 클래스인 WeatherData를 생성시에 주입받아 옵저버로 등록 합니다.

그리고, Observer 인터페이스에서 정의한 `update` 메서드를 재정의하여, Subject 클래스에서 받고싶은 정보들을 멤버 변수에 대입해 줍니다.

🗅 **class ForecastDisplay**
```java
public class ForecastDisplay implements Observer,DisplayElement{  
  
    private float currentPressure = 29.9f;  
    private float lastPressure;  
    private WeatherData weatherData;  
  
    public ForecastDisplay(WeatherData _weatherData){  
        this.weatherData = _weatherData;  
        weatherData.registerObserver(this);  
    }  
  
    @Override  
    public void display() {  
        System.out.print("Forecast: ");  
        if (currentPressure > lastPressure) {  
            System.out.println("날씨가 좋아지고 있습니다");  
        } else if (currentPressure == lastPressure) {  
            System.out.println("날씨가 지속되고 있습니다");  
        } else if (currentPressure < lastPressure) {  
            System.out.println("추워지고 비가올 수 있음을 대비하세요");  
        }  
    }  
  
    @Override  
    public void update(float _temp, float _humidity, float _pressure) {  
        lastPressure = currentPressure;  
        currentPressure = _pressure;  
        display();  
    }  
}
```

🗅 **class CurrentConditionsDisplay**
```java
public class CurrentConditionsDisplay implements Observer, DisplayElement{  
  
    private float temperature;  
    private float humidity;  
    private WeatherData weatherData;  
  
    public CurrentConditionsDisplay(WeatherData _weatherData){  
        this.weatherData = _weatherData;  
        weatherData.registerObserver(this);  
    }  
  
    @Override  
    public void update(float _temperature, float _humidity, float _pressure) {  
        this.temperature = _temperature;  
        this.humidity = _humidity;  
        display();  
    }  
    @Override  
    public void display() {  
        System.out.println("현재 상태 : 온도: "+temperature+"F, 습도: "+humidity+"%");  
    }  
  
}
```


위와 같이 하면, Subject 클래스에서 구현한 `notifyObservers`가 호출될 때, 각 Observer들에게 정보가 전달이 됩니다.

#### Main 함수에서 실행

이제 각 `Subject`와 `Observer`들을 구현했으니, Main 함수에서 실행해 봅니다.

🗅 **class Main**
```java

public class Main {  
    public static void main(String[] args) {  
        WeatherData weatherData = new WeatherData();  
  
        CurrentConditionsDisplay currentConditionsDisplay = new CurrentConditionsDisplay(weatherData);   
        ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);  

  
        weatherData.setMeasurements(80f,65f,30.45f);  
    }  
}
```

![](/images/Pasted%20image%2020240302010538.png)

다음과 같이 각`Observers`를 등록하고 난 후, `Subject`클래스의 `setMeasurements` 함수가 호출되고, 출력값이 정상적으로 나오는것을 확인할 수 있습니다.

방금 구현한 방식은 객체를 등록하면 모든 객체에게 정보를 `push`해주는 방식입니다. 즉, 정보를 뿌리는 주체는 `Subject`클래스 인거죠.

하지만, 다음과 같은 요구사항이 생길 수 있습니다. "나는 내가 필요한 정보만을 가져가고 싶어"

해당 요구사항을 만족시키기 위해 사용할 수 있는 방식이 `pull`방식 입니다.


## 예제 - push 방식

`pull`방식은 데이터가 바뀌면 옵저버의`update()` 함수를 호출해서 옵저버에 새로운 데이터를 보냅니다.

이제 구현해 볼것은 데이터가 바뀌었다는 알림을 옵저버가 받았을 때, `Subject`에 있는 게터 메소드를 호출해서 필요한 값을 당겨오도록 만듭니다.

### 기존 클래스에서 수정

우선, `Subject` 클래스인 WeatherData를 바꿔 줍니다.
`push`방식에서는 update 함수의 인자로 데이터들을 넘겨줬지만, `pull`방식에서는 단순히 update 함수만 호출을 해줍니다.

물론, override method 가 바뀜에 따라 기본 정의인 interface 에서도 update 함수의 정의를 바꿔줘야 합니다.
🗅 **class WeatherData**
```java
@Override  
public void notifyObservers() {  
    for(Observer observer : observerList){  
        observer.update();  
    }  
}
```

🗅 **interface Observer**
```java
public interface Observer {  
  
    //옵저버들이 가질 update 메소드  
    //온도, 습도, 기압 정보를 update    
    public void update();
}
```

그다음, 각 옵저버의 `update` 메서드가 호출될 때, `Subject`의 getter 를 활용하여 정보를 가져 오도록 바꿉니다.

🗅 **class CurrentConditionsDisplay**

```java  
@Override  
public void update() {  
    this.temperature = weatherData.getTemperature();  
    this.humidity = weatherData.getHumidity();  
    display();  
}
```

이렇게 바꾸면 `pull` 방식으로 구현이 완료가 됩니다.

### Main 함수 실행

🗅 **class Main**
```java
public class Main {  
    public static void main(String[] args) {  
        WeatherData weatherData = new WeatherData();  
  
        ForecastDisplay currentConditionsDisplay = new ForecastDisplay(weatherData); 
  
        weatherData.setMeasurements(80f,65f,30.45f);  
    }  
}
```

![실행결과](/images/Pasted%20image%2020240302131201.png)

정상적으로 정보를 가져오는것을 확인할 수 있습니다.

`push`방식과 다르게 원하는 데이터만을 가져오는 것을 확인할 수 있습니다.

