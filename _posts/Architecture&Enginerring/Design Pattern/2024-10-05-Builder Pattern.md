---
layout: post

title: "[Design Pattern] 빌더 패턴(Builder Pattern)"

categories:
  -  Architecture&Engineering
  
tags:
  - Java
  - Head-First-Design-Pattern
  - Book

toc: true
toc_sticky: true

date: 2024-10-05
last_modified_at: 2024-10-05
---

## 빌더 패턴 이란?

> 빌더 패턴은 복잡한 객체의 생성 과정과 표현 방법을 분리하여 다양한 구성의 인스턴스를 만드는 패턴 입니다.
>
> 선택적인 속성을 유연하게 받아서, 다양한 타입의 인스턴스를 생성할 수 있어서, 선택적 매개변수가 많은 경우 사용 하게 됩니다.


### 패턴의 장점

- 복합 객체의 생성 과정을 캡슐화 합니다.
- 여러 단계와 다양한 절차를 거쳐 객체를 만들 수 있습니다.
- 제품의 내부 구조를 클라이언트로부터 보호할 수 있습니다.
- 클라이언트는 추상 인터페이스만 볼 수 있기에 제품을 구현한 코드를 쉽게 바꿀 수 있습니다.

### 패턴의 단점

- 팩토리패턴은 한방에 객체를 만들어주는 반면, 빌더 패턴은 사용자가 어떤 구성요소가 있는지에 대해 더 많이 알아야 합니다.
- 생성자에서 객체를 생성하는것 보다 성능이 떨어집니다.

## 예제

빌더패턴을 통하여, 휴가 일정을 정하는 예제를 만들어 봅니다. 휴가는 `도시에서 즐기는 휴가` 와 `야외에서 즐기는 휴가` 두종류로 나뉘고, 해당 휴가를 만드는 빌더 역시 2가지로 나뉘게 됩니다.

우선, `숙소`, `예약`, `휴가` 의 객체를 만듭니다. `숙소`는 여러가지가 나올 수 있기에, 추상클래스로 공통점을 묶습니다.

### 예약 만들기

우선, 예약 이라는 객체를 만듭니다. 예약의 대한 정보를 담고 있습니다.

🗅 **<span style="color: #c03a92">class Reservation</span>**

```java
package src.builder.elements;  
  
import java.time.LocalDate;  
  
public class Reservation {  
    private LocalDate arrivalDate;  
    private int nights;  
  
    public void setArrivalDate(int _year, int _month, int _day) {  
        this.arrivalDate = LocalDate.of(_year, _month, _day);  
    }  
    public LocalDate getArrivalDate() {  
        return this.arrivalDate;  
    }  
    public void setNights(int _nights) {  
        this.nights = _nights;  
    }  
    public int getNights() {  
        return this.nights;  
    }  
}
```

### 숙소 만들기

다음에는 예약을 활용해서, 숙소에 대한 정보를 만듭니다. 숙소는 야회 활동시에는 `Tent` 를 활용하게 되고, 도시에서 활동시에는 `Hotel`을 활용하게 됩니다. `Tent`와 `Hotel`을 공통적으로 추상화한 `숙소`라는 추상 클래스를 정의 합니다.

🗅 **<span style="color: #c03a92">abstract class Accommodation</span>**

```java
package src.builder.elements;  
  
//숙소  
public abstract class Accommodation {  
  
    protected String name = null;  
    private Reservation reservation = null;  
  
    public void setReservation(Reservation _reservation) {  
        this.reservation = _reservation;  
    }  
    public Reservation getReservation() {  
        return this.reservation;  
    }  
    public abstract String getLocation();  
  
    @Override  
    public String toString() {  
        StringBuffer display = new StringBuffer();  
        display.append("당신은 " + name + "에 머물고 있습니다.");  
        if (this.reservation != null) {  
            display.append("\n도착일: " + reservation.getArrivalDate() +  
                    ", " + reservation.getNights() + "박 동안 예약이 되어 있습니다.");  
        }  
        if (this.getLocation() != "") {  
            display.append(" 위치는 " + this.getLocation() + "입니다.");  
        }  
        display.append("\n");  
        return display.toString();  
    }  
  
}
```

추상클래스를 정의 한 후, 추상 클래스를 상속받아 `Tent`와 `Hotel`의 구현 클래스를 정의 합니다.

🗅 **<span style="color: #c03a92">class Tent</span>**

```java
package src.builder.elements;  
  
public class Tent extends Accommodation {  
    int siteNumber;  
    public Tent() {  
        this.name = "텐트";  
    }  
    public Tent(String _name) {  
        this.name = _name;  
    }  
    public void setSiteNumber(int _n) {  
        this.siteNumber = _n;  
    }  
    public int getSiteNumber() {  
        return this.siteNumber;  
    }  
    public String getLocation() {  
        if (siteNumber == 0) return "";  
        else return "사이트 번호 " + this.siteNumber;  
    }  
}
```

🗅 **<span style="color: #c03a92">class Hotel</span>**

```java
package src.builder.elements;  
  
public class Hotel extends Accommodation {  
    int roomNumber;  
    public Hotel() {  
        this.name = "호텔";  
    }  
    public Hotel(String name) {  
        this.name = name;  
    }  
    public void setRoomNumber(int n) {  
        this.roomNumber = n;  
    }  
    public int getRoomNumber() {  
        return this.roomNumber;  
    }  
    public String getLocation() {  
        if (roomNumber == 0) return "";  
        else return "방번호:  " + this.roomNumber;  
    }  
}
```

### 휴가 객체 만들기

빌더에서 최종적으로 얻을 `휴가`라는 객체를 정의 합니다. 해당 객체는 여러개의 `숙소`와 여러가지의 `이벤트` 들을 담을 수 있도록 설계 합니다.

🗅 **<span style="color: #c03a92">class Vacation</span>**

```java
package src.builder.elements;  
  
import java.util.ArrayList;  
import java.util.List;  
  
public class Vacation {  
    private String name = null;  
    private List<Accommodation> accommodations = new ArrayList<>();  
    private List<String> events = new ArrayList<String>();  
  
    public void setName(String _name) {  
        this.name = _name;  
    }  
    public void setAccommodations(List<Accommodation> _accommodations) {  
        this.accommodations = _accommodations;  
    }  
    public void setEvents(List<String> _events) {  
        this.events = _events;  
    }  
    public String toString() {  
        StringBuffer display = new StringBuffer();  
        display.append("---- " + this.name + " ----\n");  
        for (Accommodation a : accommodations) {  
            display.append(a);  
        }  
        for (String e : events) {  
            display.append(e + "\n");  
        }  
        return display.toString();  
    }  
}
```

### 빌더 만들기

이제 요소들을 정의 하였으니, 이 요소들을 조합하여 휴가를 만들어 줄, 빌더를 만들도록 합니다. 빌더 역시 `도시에서 즐기는 휴가를 만들어주는 빌더`, `야외에서 즐기는 휴가를 만들어주는 빌더` 로 나뉘게 됩니다. 그래서 `휴가 빌더`라는 추상클래스를 만들어 줍니다.

🗅 **<span style="color: #c03a92">abstract class VacationBuilder</span>**

```java
package src.builder.builder;  
  
import src.builder.elements.Accommodation;  
import src.builder.elements.Vacation;  
  
import java.util.ArrayList;  
import java.util.List;  
  
public abstract class VacationBuilder {  
    protected String name = null;  
    protected List<Accommodation> accommodations = new ArrayList<Accommodation>();  
    protected List<String> events = new ArrayList<String>();  
  
    public abstract VacationBuilder addAccommodation();  
    public abstract VacationBuilder addAccommodation(String _name);  
    public abstract VacationBuilder addAccommodation(String _name, int _year, int _month, int _day, int _nights, int _location);  
    public abstract VacationBuilder addEvent(String _event);  
  
    public Vacation getVacation(){  
        Vacation vacation = new Vacation();  
        vacation.setName(name);  
        vacation.setAccommodations(accommodations);  
        vacation.setEvents(events);  
        return vacation;  
    }   
}
```

해당 추상 클래스는 `숙소를 추가한다` 와 `이벤트를 추가한다` 라는 기능을 하게 됩니다. 그리고 마지막으로 `getVacation()`을 통해서, 만들어진 `휴가` 객체를 반환 해줍니다.

이제 이 추상클래스를 상속받아 `도시에서 즐기는 휴가를 만들어주는 빌더`, `야외에서 즐기는 휴가를 만들어주는 빌더` 를 구현해 봅시다.

🗅 **<span style="color: #c03a92">class CityVacationBuilder</span>**

```java
package src.builder.builder;  
  
import src.builder.elements.Hotel;  
import src.builder.elements.Reservation;  
  
public class CityVacationBuilder extends VacationBuilder{  
  
    public CityVacationBuilder(){  
        this.name = "씨티 투어 빌더";  
    }  
    @Override  
    public VacationBuilder addAccommodation() {  
        this.accommodations.add(new Hotel());  
        return this;  
    }  
  
    @Override  
    public VacationBuilder addAccommodation(String _name) {  
        this.accommodations.add(new Hotel(name));  
        return this;  
    }  
  
    @Override  
    public VacationBuilder addAccommodation(String _name, int _year, int _month, int _day, int _nights, int _location) {  
        Reservation reservation = new Reservation();  
        reservation.setArrivalDate(_year, _month, _day);  
        reservation.setNights(_nights);  
  
        Hotel hotel = new Hotel(name);  
        hotel.setReservation(reservation);  
        hotel.setRoomNumber(_location);  
        this.accommodations.add(hotel);  
        return this;  
    }  
  
    @Override  
    public VacationBuilder addEvent(String _event) {  
        this.events.add(_event + " 공연을 관람합니다.");  
        return this;  
    }  
}
```

🗅 **<span style="color: #c03a92">class OutdoorsVacationBuilder</span>**

```java
package src.builder.builder;  
  
import src.builder.elements.Reservation;  
import src.builder.elements.Tent;  
  
public class OutdoorsVacationBuilder extends VacationBuilder{  
    public OutdoorsVacationBuilder() {  
        this.name = "야외 휴가 빌더";  
    }  
    public VacationBuilder addAccommodation() {  
        this.accommodations.add(new Tent());  
        return this;  
    }  
    public VacationBuilder addAccommodation(String _name) {  
        this.accommodations.add(new Tent(_name));  
        return this;  
    }  
    public VacationBuilder addAccommodation(String _name, int _year, int _month, int _day, int _nights, int _location) {  
        Reservation reservation = new Reservation();  
        reservation.setArrivalDate(_year, _month, _day);  
        reservation.setNights(_nights);  
  
        Tent tent = new Tent(name);  
        tent.setReservation(reservation);  
        tent.setSiteNumber(_location);  
        this.accommodations.add(tent);  
        return this;  
    }  
    public VacationBuilder addEvent(String event) {  
        this.events.add("Hike: " + event);  
        return this;  
    }  
}
```


### 메인에서 실행

빌더들을 만들고, 생성하여 `휴가`일정을 정의 합니다. 여기서 원하는 일정이 있다면, 생성자를 오버라이딩 해서 추가하는것이 아닌, `.addAccommodation()` 이나 `.addEvent()`로 객체에 속성을 부여하면 됩니다.

🗅 **<span style="color: #c03a92">class Main</span>**

```java
package src.builder;  
  
import src.builder.builder.CityVacationBuilder;  
import src.builder.builder.OutdoorsVacationBuilder;  
import src.builder.builder.VacationBuilder;  
import src.builder.elements.Vacation;  
  
public class Main {  
    public static void main(String[] args) {  
        VacationBuilder cityVacationBuilder = new CityVacationBuilder();  
        Vacation cityVacation = cityVacationBuilder  
                .addAccommodation("그랜드 파카디안", 2020, 8, 1, 5, 0)  
                .addAccommodation("호텔 커맨더", 2020, 8, 6, 2, 0)  
                .addEvent("태양의 서커스")  
                .getVacation();  
        System.out.println(cityVacation);  
  
        VacationBuilder outdoorVacationBuilder = new OutdoorsVacationBuilder();  
        Vacation outdoorVacation =outdoorVacationBuilder  
                .addAccommodation("2인용 텐트", 2020, 7, 1, 5, 34)  
                .addEvent("해변")  
                .addAccommodation("2인용 텐트")  
                .addEvent("산")  
                .getVacation();  
        System.out.println(outdoorVacation);  
    }  
}
```

![](/images/Pasted%20image%2020241005222940.png)

## 정리

빌더 패턴은 제가 안드로이드 앱 개발을 했을 때 많이 접해볼 수 있었습니다. 안드로이드의 통신 라이브러인 `OkHttp` 나 `Dialog`를 만들 때 선택적인 속성들을 빌더 패턴을 통하여 생성 하였습니다.

처음 빌더 패턴을 접했을때는, 계속 .add() .add() 하는 문법이 어떻게 구성되어 있는지 감이 잡히지 않았습니다. 하지만 간단하게 만들어보니, 안드로이드의 라이브러리도 어떤식으로 구성되어 있는지 조금은 알 것 같습니다.

빌더패턴은 생성자를 오버라이딩 할 필요 없이, 함수 호출만으로 객체에게 상태를 부여할 수 있다는 장점이 있지만, 상속구조가 복잡해지고, 생성자보다 성능이 내려간다는 단점이 있습니다. 즉, 불필요하게 사용하기 보다는 여러가지 상태를 가진 객체를 많이 만들 때 유용한 것 같습니다.

