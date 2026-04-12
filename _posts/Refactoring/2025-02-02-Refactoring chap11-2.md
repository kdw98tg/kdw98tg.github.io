---
title: "[Refactoring] 리팩터링 2판 Chapter.11 part 2 (API 리팩터링)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2025-02-02
last_modified_at: 2025-02-02
---

## 세터 제거하기

세터 메서드가 있다고 함은 필드가 수정될 수 있다는 뜻입니다. 객체 생성 후에는 수정되지 않길 원하는 필드라면 세터를 제공하지 않았을 것입니다.

해당 리팩터링은 세터를 쓰지 않으면 세터를 지워라 라는 내용입니다. 너무나 당연한 내용이라 뺄까 했지만 새로 알게된 부분이 있어서 기술하였습니다.

생성자에서 초기화 할때, 세터 함수를 호출하는 부분이 있습니다. 물론 저도 그렇게 사용하고 있습니다만, 저자는 해당 부분에서 **한번만 세팅해주는 함수라면 세터를 제거하여 다시는 세팅되지 않을 수 있도록 못을 박아 의도를 명확하게 하라 라고 하였습니다.** 지금 생각해보니, 코드가 깔끔하지 않을거 같아서 전부 함수로 캡슐화해서 사용했는데, 어차피 한번만 세팅할거라면 함수로 만들지 않는것이 의도를 명학하게 하는 방법이라는 생각이 듭니다.

### 절차
1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다. 그런 다음 생성자 안에서 적절한 세터를 호출한다.
2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.
3. 세터 메서드를 인라인 한다. 가능하다면 해당 필드를 불변으로 만든다.
4. 테스트한다.

## 생성자를 팩터리 함수로 바꾸기

많은 객체 지향 언어에서 제공하는 생성자는 객체를 초기화하는 특별한 용도의 함수입니다. 실제로 새로운 객체를 생성할 때면 주로 생성자를 호출하게 됩니다. 하지만 만약, 건물을 생성하는데, 건물의 타입을 정하는 `enum`에 의해서 객체가 달라진다면 어떨까요? 클라이언트는 해당 건물의 `enum`도 알고 있어야 하고, 어떤 건물이 리턴되는지도 알고 있어야 합니다. 이럴 때, 생성자를 팩터리 함수로 바꿔서 호출하도록 하면 좋습니다. (JS에 대한 이해도가 높지 않으므로 해당 예제는 CS로 작성하였습니다.)

### 절차

1. 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래의 생성자를 호출한다.
2. 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다.
3. 하나씩 수정할 때마다 테스트한다.
4. 생성자의 가시 범위가 최소가 되도록 제한한다.

### 예제

```cs
public class BuildingObject
{
	private BuildingType type;
	public BuildingObject(BuildingType _type)
	{
		this.type = _type;
	}
}

//클라이언트
BuildingObject aTypeBuilding = new BuildingObject(BuildingType.A);
BuildingObject bTypeBuilding = new BuildingObject(BuildingType.B);
```

위와같이 호출하는것 대신에 다음과 같이 호출합니다.

```cs
public class BuildingObject
{
	private BuildingType type;
	private BuildingType(BuildingType _type)
	{
		this.type = _type
	}

	public static BuildingObject CreateATypeBuilding(){
		return new BuildingType(BuildingType.A);
	}
	public static BuildingObject CreateBTypeBuilding(){
		return new BuildingType(BuildingType.B);
	}
}

BuildingObject aTypeBuilding = BuildingObject.CreateATypeBuilding();
BuildingObject bTypebuilding = BuildingObject.CreateBTypeBuilding();
```

## 오류 코드를 에외로 바꾸기

해당 파트는 얼마전에 예외에 대해서 생각하다가 책을 봤는데 공감되는 내용이 많아서 저자의 말을 많이 인용하였습니다.

1세대 프로그래밍을 하면 당시에는 예외 대신에 오류 코드를 사용한게 보편적이었습니다. 함수를 호출하면 언제든 오류가 반환될 수 있었고, 그래서 오류 코드 검사를 빼먹으면 안됐습니다. 

예외는 프로그래밍 언어에서 제공하는 독립적인 오류 처리 메커니즘입니다. 오류가 발견되면 예외를 던집니다. 그러면 적절한 예외 핸들러를 찾을 때까지 콜스택을 타고 위로 전파됩니다. 예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별해 콜스택 위로 던지는 일을 신경 쓰지 않아도 됩니다. 예외에는 독자적인 흐름이 있어서 프로그램의 나머지에서는 오류 발생에 따른 복잡한 상황에 대처하는 코드를 작성하거나 읽을 일이 없게 해줍니다.

예외는 정교한 메커니즘이지만 대다수의 다른 정교한 메커니즘과 같이 정확하게 사용할 때만 최고의 효과를 냅니다. 예외는 정확히 예상 밖의 동작일 때만 쓰여야 합니다. 달리 말하면 프로그램의 정상 동작 범주에 들지 않는 오류를 나타낼 때만 쓰여야 합니다. 괜찮은 법칙이라면, 예외를 던지는 코드를 프로그램 종료 코드로 바꿔도 프로그램이 여전히 정상 작동할지를 따져보는 것입니다. 정상 동작하지 않을 것 같다면 예외를 사용하지 말라는 신호입니다. 예외 대신 오류를 검출하여 프로그램을 정상 흐름으로 되돌려 놔야 합니다.

### 절차
1. 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성합니다.
2.  테스트한다.
3. 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.
4. 정적 검사를 수행한다.
5. catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대체하고 그렇지 않은 예외는 다시 던진다.
6. 테스트한다.
7. catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대체하고 그렇지 않은 예외는 다시 던진다.
8. 테스트한다.
9. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다.
10. 모두 수정했다면 그 오류 코드를 콜스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다.

## 예외를 사전확인으로 바꾸기

예외라는 개념은 프로그래밍 언어의 발전에 의미있는 한걸음이었습니다. 오류 코드를 연쇄적으로 전파하던 긴 코드를 예외로 바꿔 깔끔히 제거할 수 있게 되었습니다. 하지만 좋은 것들이 늘 그렇듯, 예외도 과용되면 좋은게 아니게 됩니다. 예외(뜻밖의 오류)는 말 그대로 예외적으로 동작할 때만 쓰여야 합니다. **함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 해야 합니다.**

### 절차
1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. catch 블록의 코드를 조건문의 조건절중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다.
2. catch 블록에 어서션을 추가하고 테스트한다.
3. try문과 catch 블록을 제거한다.
4. 테스트한다.


### 예제

데이터베이스 연결 같은 자원들을 관리하는 자원 풀 클래스가 있다고 가정합니다. 만약 자원이 바닥난다면 다시 채우는 코드를 작성한다고 가정합니다.

```java
public Resource get(){
	Resource result;
	try{
		result = available.pop();
		allocated.add(result);
	}
	catch (NoSuchElementException e){
		result = Resource.create();
		allocated.add(result);
	}
	return result;
}

private Deque<Resource> available;
private List<Resource> allocated;
```

위 코드에서 만약 풀의 자원이 고갈되었다면, `pop()`이 실행될 때 예외가 발생할 수 있습니다. 하지만, 풀에서 자원이 고갈되는것은 예외가 아니라 충분히 예상 가능한 상황입니다. 이럴때는 try~catch 블록을 사용하면 안됩니다.

```java
public Resource get(){
	Resource result;
	if(available.isEmpty()){
		result = Resource.create();
		allocated.add(result);
	}
	else{
		try{
			result = available.pop();
			allocated.add(result);
		}catch(NoSuchElementException e){
			
		}
	}
	return result;
}
```

위처럼 바꿔서 사용해야 합니다. 근데 이제 보니까 catch 절은 사용하지 않는군요. 여기서는 로깅을 하거나 어서션을 추가하여 예상치못한 상황에 대비 합니다.

```java
public Resource get(){
	Resource result;
	if(available.isEmpty()){
		result = Resource.create();
		allocated.add(result);
	}
	else{
		try{
			result = available.pop();
			allocated.add(result);
		}catch(NoSuchElementException e){
			throw new AssertionError("도달 불가");
		}
	}
	return result;
}
```

여기까지 하고보니, 로깅하는 로직이 아니라면, 굳이 try~catch 절도 필요없어 보입니다. 테스트를 거친 후 어서션을 통과했다면 try~catch 블록도 삭제해 줍니다.

```java
public Resource get(){
	Resource result;
	if(available.isEmpty()){
		result = Resource.create();
		allocated.add(result);
	}
	else{
		result = available.pop();
		allocated.add(result);
	}
	return result;
}
```

여기까지하면, 리팩터링의 끝입니다.