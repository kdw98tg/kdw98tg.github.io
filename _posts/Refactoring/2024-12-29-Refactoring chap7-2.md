---
title: "[Refactoring] 리팩터링 2판 Chapter.07 part 2 (캡슐화)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-12-29
last_modified_at: 2024-12-29
---

## 기본형을 객체로 바꾸기

개발 초기에는 단순한 정보를 숫자나 문자열 같은 간단한 데이터 항목으로 표현할 때가 많습니다. 전화번호를 처음에는 문자열로 표현하는 것이 있습니다. 하지만 개발을 진행하고, 서비스가 고도화 되면서 포맷팅이나 지역 코드 추출같은 특별한 동작이 추가되는 경우가 있습니다. 기능이 추가될수록, 코드를 바로 이해하는데 시간이 필요하게 됩니다.

그렇기에 저자는 단순한 출력 이상의 기능이 필요해지는 순간 그 데이터를 표현하는 전용 클래스를 정의한다고 합니다. 나중에 특별한 동작이 추가되는 상황을 고려한 것입니다. 숙련된 개발자들은 이미 많이 사용하고 있는 기법이라고 합니다.

### 절차
1. 아직 변수를 캡슐화하지 않았다면 캡슐화 한다.
2. 단순한 값 클래스를 만든다. 생성자는 기존 값을 인수로 받아서 저장하고, 이 값을 반환하는 게터를 추가한다.
3. 정적 검사를 수행한다.
4. 값 클래스의 인스턴스를 새로 만들어서 필드에 저장하도록 세터를 수정한다. 이미 있다면 필드의 타입을 적절히 변경한다.
5. 새로 만든 클래스의 게터를 호출한 결과를 반환하도록 게터를 수정한다.
6. 테스트한다.
7. 함수 이름을 바꾸면 원본 접근자의 동작을 더 잘 드러낼 수 있는지 검토한다.

### 예제

주문중에서, 우선순위가 일정 등급 이상인 주문들의 갯수를 뽑아내는 예제입니다.

우선, Order 클래스와 Priority 클래스를 정의 합니다.

```js
class Order {
    constructor(data) {
        this.priority = data.priority;
    }
    get priority() {
        return this._priority;
    }
    set priority(aString) {
        this._priority = aString;
    }
}
```

```js
class Priority {
    constructor(value) {
        this.value = value;
    }
}
```

그리고 클라이언트에서 우선순위가 `normal` 보다 높은 주문의 갯수를 뽑아내는 코드를 작성합니다.

```js
highPriorityCount = orders.filter(o => o.priority.value > 'normal').length;
```

이렇게 하면 동작은 하겠지만, 조건이 추가되어 식이 복잡해지면, 해당 코드를 해석하는데 시간이 필요할 것입니다. 따라서 Priority 클래스에 `isHigherThan`이라는 함수를 추가하여 해당 조건을 캡슐화 합니다.

```js
class Priority {
    constructor(value) {
        this.value = value;
    }
    isHigherThan(otherPriority) {
        return this.value > otherPriority.value;
    }
}
```

캡슐화를 하고 난 후, 클라이언트에서의 코드는 다음과 같이 변합니다.

```js
highPriorityCount = orders.filter(o => o.priority.isHigherThan(new Priority('normal'))).length;
```

이렇게 하면 해당 조건이 어떤것인지 한눈에 알아볼 수 있게 됩니다.

## 임시 변수를 질의 함수로 바꾸기

어떤 모델 클래스 안에서 결과값을 반환할 때, 다시 참조할 목적으로 임시 변수를 사용하곤 합니다. 임시 변수를 사용하면, 한번만 계산하고 계산한 결과값을 참조하기 때문에 코드가 반복되는걸 줄이고, 값의 의미를 변수명으로 설명할 수 있기 때문에 유용합니다. 이러한 장점을 더 살리기 위해서, 해당 임시 변수를 함수로 만들어 사용합니다. (함수로 만들어 사용하는 편이 더 나을때가 많습니다.)

또한, 함수로 만들어 놓으면 당연한 말이겠지만, 같은 계산을 하는 다른 부분에도 재사용할 수 있습니다. 또한, 계산 방법이 바뀌면, 함수만 수정하면 되기 때문에 관리포인트도 줄어들게 됩니다.

### 절차
1. 변수가 사용되기 전에 값이 확실히 결정되는지, 변수를 사용할 때마다 계산 로직이 매번 다른 결과를 내지는 않는지 확인한다.
2. 읽기전용으로 만들 수 있는 변수는 읽기전용으로 만든다.
3. 테스트한다.
4. 변수 대입문을 함수로 추출한다.
5. 테스트한다.
6. 변수 인라인하기로 임시 변수를 제거한다.

### 예제

예제는 간단하게 주문 클래스 안에서 값과 수량을 받아서 최종 가격을 계산하는 코드를 가져왔습니다.

우선, 임시 변수를 사용하여 `totalPrice`를 가져오는 코드 입니다.

```js
class Order {

    constructor(quantity, price) {
        this.quantity = quantity;
        this.price = price;
    }

    get totalPrice() {
        const basePrice = this.quantity * this.price;

        if (basePrice > 1000) {
            return this.basePrice * 0.95;
        }

        return basePrice * 0.98;
    }
}
```

`totalPrice` 라는 함수 안에서 basePrice라는 임시 변수를 사용하여 계산을 하는 방식입니다. 하지만, 이렇게 하게되면 basePrice를 구하는 공식이 달라지거나, basePrice를 필요로 하는 함수가 많아질수록 관리포인트가 늘어나게 됩니다. 따라서 basePrice를 구하는 식을 함수로 추출합니다.

```js
class Order {
    constructor(quantity, price) {
        this.quantity = quantity;
        this.price = price;
    }

    get basePrice() {
        return this.quantity * this.price;
    }

    get totalPrice() {
        if (this.basePrice > 1000) {
            return this.basePrice * 0.95;
        }
        return this.basePrice * 0.98;
    }
}
```

이렇게하면, basePrice를 구하는 식을 다른 함수에서 참조하게 되므로, 관리하기가 훨씬 쉬워집니다.
