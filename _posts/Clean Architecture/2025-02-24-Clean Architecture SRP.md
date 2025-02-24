---
title: "[Clean Architecture] Clean Architecture SRP(Single Responsibility Principle단일책임의 원칙)"

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

## SRP 의 오해와 정의
단일책임의 원칙이라고 하면 보통 이렇게 이야기를 합니다. "모듈은 단 하나의 일만 해야 한다."

하지만 이것은 반은 맞고 반은 틀린 이야기 입니다. 그래서 저자는 SOLID 원칙 중에서 그 의미가 가장 잘 전달되지 못한 원칙이 바로 이 SRP 라고 합니다.

위에서 통상적으로 알고 있는 "모듈은 단 하나의 일만 해야 한다는" 커다란 함수를 작은 함수들로 리팩터링하는 더 저수준에서 사용되는 의미입니다. 게다가 이 원칙은 SOLID 원칙이 아닐 뿐더러 SRP도 아닙니다.

따라서 제대로 된 SRP의 정의는 다음과 같이 말할 수 있습니다.

>단일 모듈은 변경의 이유가 하나, 오직 하나뿐이어야 한다.

또는 이 클래스를 변경하는 주체 기준으로 다음과 같이 말할 수 있습니다.

> 하나의 모듈은 하나의, 오직 하나의 사용자 또는 이해관계자에 대해서만 책임져야 한다.

위 두개를 합치면 다음의 의미로 해석할 수 있습니다.

> 하나의 모듈은 하나의, 오직 하나의 액터에 대해서만 책임져야 한다.

## SRP 위반 사례

직원 정보를 담당하는 Employee 클래스에는 4가지 주요 메서드가 존재한다고 가정합니다.

```cs
public class Employee
{
	void SaveDatabase();//변경된 정보를 저장
	void CalculatePay();//회계팀에서 급여를 계산하는 메서드
	void CalculateExtraHour();//초과근무시간을 계산하는 메서드
	void ReportHours();//인사팀에서 근무시간을 계산하는 메서드
}
```

여기서, `CalculatePay()`와 `ReportHours()` 메서드에서 초과 근무 시간을 계산하기 위해 `CalculateExtraHour()` 메서드를 공유하여 사용하고 있다고 가정합니다.

```cs
//SRP를 위반한 예시
public class Employee
{
    private string name;
    private string position;

    public Employee(string _name, string _position)
    {
        this.name = _name;
        this.position = _position;
    }

    //초과 근무 시간을 계산하는 메서드
    public void CalculateExtraHour()
    {
        //...
    }

    //급여를 계산하는 메서드 (회계팀에서 사용)
    public void CalculatePay()
    {
        //...
        CalculateExtraHour();
        //...
    }

    //근무시간을 계산하는 메서드 (인사팀에서 사용)
    public void ReportHours()
    {
        //...
        CalculateExtraHour();
        //...
    }

    //변경된 정보를 DB에 저장하는 메서드 (기술팀에서 사용)
    public void SaveDatabase()
    {
        //...
    }
}
```

여기서, 문제가 발생합니다. 회계팀에서 급여를 계산하는 기존의 방식을 새로 변경하여, 코드에서 초과 근무 시간을 계산하는 메서드인 `CalculateExtraHour()`의 알고리즘의 업데이트가 필요해졌습니다.

그래서 개발팀에서 회계팀의 요청을 바꿨는데, `CalculatePay()`함수와 `ReportHours()`함수에 영향을 주어 예상치 않은 결과를 반환하게 되었습니다.

**바로 이 상황이 SRP에 위배된 상황 입니다. 현재 Employee 클래스에서는 회계팀, 인사팀, 기술팀 이렇게 3개의 엑터에 대한 책임을 한꺼번에 가지고 있기 때문입니다.**

(여기서 액터는 시스템을 수행하는 역할을 하는 요소로써, 시스템을 이용하는  사용자, 하드웨어 혹은 외부 시스템이 될 수 있음)

그래서, 해당 문제를 해결하는데에 `Facade Pattern`을 사용할 수 있습니다. Employee Facade를 만들고 해당 클래스는 나누어진 클래스들의 로직을 참조할 수 있도록 해주는 중재자의 역할입니다.

```cs
public class EmployeeFacade
{
    private string name;
    private string position;

    public EmployeeFacade(string _name, string _position)
    {
        this.name = _name;
        this.position = _position;
    }

    public void CalculatePay()
    {
        //...
        new PayCalculator().CalculatePay();
        //...
    }

    public void ReportHours()
    {
        //...
        new HourReporter().ReportHours();
        //...
    }

    public void SaveEmployees()
    {
        //...
        new EmployeeSaver().SaveDatabase();
        //...
    }
}
```

```cs
public class EmployeeSaver
{
    public void SaveDatabase()
    {
        
    }
}
```

```cs
//인사팀에서 사용되는 전용 클래스
public class HourReporter
{
    public void CalculateExtraHour()
    {
        //...
    }

    public void ReportHours()
    {
        //...
        CalculateExtraHour();
        //...
    }
}
```

```cs
//회계팀에서 사용되는 전용 클래스
public class PayCalculator
{
    public void CalculateExtraHour()
    {

    }

    public void CalculatePay()
    {
        //...
        CalculateExtraHour();
        //...
    }
}
```

이런식으로 클래스를 나눠서, 부수효과를 일으키지 않도록, 단일 책임을 질 수 있도록 수정하였습니다.

## 결론

널리 알려진 단일 책임 원칙의 해석은 단순히 모듈이 하나의 역할만 수행하도록 제한하는 데에 초점을 맞춥니다. 그러나 그 근본적인 이유는 바로 모듈 변경의 원인이 되는 액터에 있습니다. 각 액터는 시스템 내에서 고유한 목적과 동기를 가지고 행동하며, 이로 인해 발생하는 변경 요구 또한 서로 다릅니다. 만약 모듈이 여러 책임을 지게 된다면, 하나의 액터가 일으킨 변경이 다른 액터의 요구와 충돌하거나 예기치 않은 부수 효과를 초래할 위험이 있습니다.

따라서, 모듈이 단 하나의 책임만을 져야 한다는 원칙은 액터의 관점에서 볼 때, 변경의 원인을 명확히 분리하고 각 액터의 요구에 독립적으로 대응하기 위한 전략적 설계 원칙으로 이해될 수 있습니다.

SRP는 이름만 들었을때는 가장 적용하기 쉬운 원칙처럼 보이나, 실제로 이것을 계속 생각해가며 클래스를 작성하는것은 쉽지 않을것 같습니다. 따라서 계속 생각하며 코딩하고, 경험을 쌓는것이 중요하다고 생각됩니다.

또한, 모든 좋은 기능들이 그렇듯, SRP도 너무 남용해서 산탄총같은 설계를 해버리면 클래스 개수가 너무 많이 늘어나고, 로직을 이해하는데 어려움이 있으니 상황에 따라 적절하게 사용하는것이 중요하다고 생각됩니다.






