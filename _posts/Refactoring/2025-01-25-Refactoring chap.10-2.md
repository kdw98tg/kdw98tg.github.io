---
title: "[Refactoring] 리팩터링 2판 Chapter.10 part 2 (데이터 조직화)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2025-01-25
last_modified_at: 2025-01-25
---

## 조건부 로직을 다형성으로 바꾸기

복잡한 조건부 로직은 프로그래밍에서 해석하기 가장 난해한 대상에 속합니다. 그렇기에 개발자는 조건부 로직을 직관적으로 구조화할 방법을 고민해야 합니다.

그 방법중 하나로 `다형성`을 이용한것이 있습니다. 예를들어, `Bird`라는 클래스에 switch ~ case 문으로 `KoreaBird`, `EnglishBird`, `JapaneseBird`가 각각 정의되어 있다면, 이는 다형성으로 분리하라는 신호일 수 있습니다.

다형성은 객체지향 프로그래밍의 핵심입니다. 하지만 남용하기가 쉽습니다. 실제로 모든 조건부 로직을 다형성으로 대체해야 한다고 주장하는 사람도 있다고 합니다. 저는 여기에 동의하지 않습니다. 간단하게 처리할 수 있는 부분을 굳이 다형성으로 만들어서 구조를 복잡하게 만들 필요는 없기 때문입니다. 역시 모든것이 과유불급인것 같습니다.


### 절차
1. 다형적 동작을 표현하는 클래스들이 아직 없다면 만들어준다. 이왕이면 적합한 인스턴스를 알아서 만들어 반환하는 팩터리 함수도 함께 만든다.
2. 호출하는 코드에서 팩터리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼클래스로 옮긴다.
4. 서브클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다. 조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스 메서드로 복사한 다음 적절히 수정한다.
5. 같은 형식으로 각 조건절을 해당 서브클래스에서 메서드로 구현한다.
6. 슈퍼클래스 메서드에는 기본 동작 부분만 남긴다. 혹은 슈퍼클래스가 추상 클래스여야 한다면, 이 메서드를 추상으로 선언하거나 서브클래스에서 처리해야 함을 알리는 에러를 던진다.

### 예제
다양한 새를 키우는 친구가 있는데, 새의 종에 따른 비행속도와 깃털 상태를 알고 싶어합니다. 이 정보를 주는 프로그램을 짭니다.

```js
function plumages(bird) {
    return new Map(birds.map(bird => [bird.name, plumage(bird)]));
}

function speeds(birds) {
    return new Map(birds.map(bird => [bird.name, airSpeedVelocity(bird)]));
}

function plumage(bird) {
    switch (bird.type) {
        case "EuropeanSwallow": return "average";
        case "AfricanSwallow": return (bird.numberOfCoconuts > 2) ? "tired" : "average";
        case "NorwegianBlueParrot": return (bird.voltage > 100) ? "scorched" : "beautiful";
        default: return "unknown";
    }
}

function airSpeedVelocity(bird) {
    switch (bird.type) {
        case "EuropeanSwallow": return 35;
        case "AfricanSwallow": return 40 - 2 * bird.numberOfCoconuts;
        case "NorwegianBlueParrot": return bird.isNailed ? 0 : 10 + bird.voltage / 10;
        default: return null;
    }
}
```

case 문에 보면 새 종류마다 분기가 나뉩니다. 이것을 클래스로 추출하여 다형성으로 구현해 봅니다.

```js
function plumage(bird) {
    return createBird(bird).plumage;
}

function airSpeedVelocity(bird) {
    return createBird(bird).airSpeedVelocity;
}

function createBird(bird) {
    switch (bird.type) {
        case 'EuropeanSwallow': return new EuropeanSwallow(bird);
        case 'AfricanSwallow': return new AfricanSwallow(bird);
        case 'NorwegianBlueParrot': return new NorwegianBlueParrot(bird);
        default: return new Bird(bird);
    }
}

class Bird {
    constructor(bird) {
        Object.assign(this, bird);
    }

    get plumage() {
        return "unknown";
    }

    get airSpeedVelocity() {
        return null;
    }
}

class EuropeanSwallow extends Bird {
    get plumage() {
        return "average";
    }

    get airSpeedVelocity() {
        return 35;
    }
}

class AfricanSwallow extends Bird {
    get plumage() {
        return (this.numberOfCoconuts > 2) ? "tired" : "average";
    }

    get airSpeedVelocity() {
        return 40 - 2 * this.numberOfCoconuts;
    }
}

class NorwegianBlueParrot extends Bird {
    get plumage() {
        return (this.voltage > 100) ? "scorched" : "beautiful";
    }

    get airSpeedVelocity() {
        return (this.isNailed) ? 0 : 10 + this.voltage / 10;
    }
}
```

bird를 만들어주는 팩터리 메서드를 만들었고, 클래스를 분리하였습니다. plumage() 와 airSpeedVelocity를 서브클래스에서 정의합니다.

이렇게 하여 swtich 문을 class로 변형하고, 캡슐화된 클래스에서 오버라이드된 함수를 호출하여 정보를 가져오게 됩니다. bird 가 추가될 때, switch case문에 추가할 필요 없이 새로운 클래스를 만들고 Bird를 상속하고 오버라이드를 해주기만 하면 됩니다.

하지만, 모든 switch case 문을 다형성으로 구현하게 되면 다형성의 역할이 부모 클래스를 확장한다는 의미보다는 같은 기능만을 묶는 의미만을 가질수도 있고, 구조가 복잡해질 수 있기 때문에, 필요할때만 사용해야 합니다.

## 특이 케이스 추가하기

데이터 구조의 특정 값을 확인한 후 똑같은 동작을 수행하는 코드가 곳곳에 등장하는 경우가 더러 있는데, 흔히 볼 수 있는 중복코드중 하나입니다. 이처럼 코드베이스에서 특정 값에 대해 똑같이 반응하는 코드가 여러 곳이라면 그 반응들을 한 데로 모으는 게 효율적입니다. (캡슐화의 의미도 담고 있는듯)

특수한 경우의 공통 동작을 요소 하나에 모아서 사용하는 `특이 케이스 패턴`이라는 것이 있는데, 바로 이럴 때 적용하면 좋은 메커니즘입니다. 특이케이스에서 특정 값을 반환해야 한다면 리터럴 객체를 반환하고, 특정 동작을 해야 한다면 특정 객체에 메서드를 만들어서 반환하면 됩니다.

이 챕터를 읽는데 시간이 많이 걸렸습니다. 그 이유는 제 생각에는 굳이 새로운 객체를 반환하지 않고, `IsValid()`와 같은 함수를 캡슐화 하여 사용하면 되는거 아닌가? 라는 생각이 계속 들었기 때문입니다. 물론 특이 케이스에 특정 동작을 하는데 그 동작이 굉장히 복잡한 것이라면 객체로 만들어 관리하는것에 동의합니다. 하지만 책에 나온 예시는 단순히 정보를 가져오는것이라 이해하기가 어려웠습니다.

### 절차
1. 컨테이너에 특이 케이스인지를 검사하는 속성을 추가하고, false를 반환하게 한다.
2. 특이 케이스 객체를 만든다. 이 객체는 특이 케이스인지를 검사하는 속성만 포함하며, 이 속성은 true를 반환하게 한다.
3. 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출한다. 모든 클라이언트가 값을 직접 비교하는 대신 방금 추출한 함수를 사용하도록 고친다.
4. 코드에 새로운 특이 케이스 대상을 추가한다. 함수의 반환 값으로 받거나 변환 함수를 적용하면 된다.
5. 특이 케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 한다.
6. 테스트한다.
7. 여러 함수를 클래스 묶기나 여러 함수를 변환 함수로 묶기를 적용하여 특이 케이스를 처리하는 공통 동작을 새로운 요소로 옮긴다.
8. 아직도 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인 한다.

### 예제

고객의 정보를 보고, 미확인 고객임을 판별하는 프로그램을 짠다고 가정 합니다.

```js
class Site {
    constructor(customer) {
        this.customer = customer;
    }
    get customer() {
        return this.customer;
    }
}

class Customer {
    constructor(name) {
        this.name = name;
    }
    get name() {
        return this.name;
    }
    get billingPlan() {
        return this.billingPlan;
    }
    set billingPlan(arg) {
        this.billingPlan = arg;
    }
    get paymentHistory() {
        return this.paymentHistory;
    }
}
```

여기서 aCustomer가 `unkown customer`일때 예외를 처리합니다.

```js
const aCustomer = site.customer;
let customerName;
if (aCustomer === "unknown") customerName = "occupant";
else customerName = aCustomer.name;

const plan = (aCustomer === "unknown") ? registry.billingPlans.basic : aCustomer.billingPlan;
```

현재는 특이 케이스를 처리하는 로직이 없기 때문에, 위 코드와 같이 if문을 계속 넣어서 특이케이스를 처리해줘야 합니다.

이렇게 했을 때, 가장 큰 문제는 만약 미확인 고객의 식별 문자열이 `unkown`이 아니라 `not found`로 바뀌게 된다면 그야말로 대 제앙이 벌어지게 됩니다. 어디에 있을지 모르는 코드들을 전부 수정해줘야 하기 때문입니다. 또한, `unkown`이 아니라 개발자의 실수로 `ukon`이라고 작성하게 되었다면 이 버그를 고치는데 한참이 걸릴 것입니다. 그래서 해당 코드를 클래스에 캡슐화 합니다.

```js
class Customer {
    constructor(name) {
        this.name = name;
    }
    get name() {
        return this.name;
    }
    get billingPlan() {
        return this.billingPlan;
    }
    set billingPlan(arg) {
        this.billingPlan = arg;
    }
    get paymentHistory() {
        return this.paymentHistory;
    }
    //판별함수 추가
    get isUnkown() {
        return false;
    }
}
```

현재 판별하는 로직을 넣진 않았는데, 이 챕터를 읽으면서 여기까지 해도 되지 않나? 라는 생각이 많이 들었습니다.

일단 저자가 이야기하는대로, 미확인 전용 클래스를 만들어 처리를 해보도록 합니다. UnkownCustomer 라는 클래스를 만들어서 name을 반환할 때 특이 케이스에서 반환할 문자열을 반환하도록 합니다. 그리고 Site 클래스에서 customer를 가져올 때, `unkown`인지 판별한 후 특이케이스를 처리하도록 합니다.

```js
class Site {
    constructor(customer) {
        this.customer = customer;
    }
    get customer() {
        return (this._customer === "unknown") ? new UnknownCustomer() : this._customer;
    }
}

class UnknownCustomer {
    get name() {
        return "occupant";
    }
}
```

위와같이 하고나면 클라이언트에서는 아래와 같이 호출하게 됩니다.

```js
const aCustomer = site.customer;
const customerName = aCustomer.name;
```

조건문을 클라이언트로 노출할 필요 없이 특이케이스까지 처리된 객체를 반환하게 됩니다. 

## 어서션 추가하기

코드를 짜다보면, 그 코드에 가정이 들어가야 할때가 많습니다. 단순한 예로, 제곱근 계산은 입력이 양수일 때만 정상 작동합니다. 보통 개발자들은 이것들을 `주석`으로 남기곤 합니다.

주석은 사람들이 사용하는 언어로 이루어지며, 컴파일때도 영향을 받지 않으며 코드에 대한 이해도가 없더라도 이해할 수 있다는 장점이 있습니다. 하지만 주석의 단점은 장점에서 그대로 비롯됩니다.

주석의 단점은 사람들이 사용하는 언어로 이루어지기 때문에 컴퓨터는 이해할 수 없고, 컴파일때에 영향을 받지 않기 때문에 주석이 훼손되어도 컴파일러가 잡아주지 못하고, 코드에 대한 이해없이 이해할 수 있다는 점에서 본래의 의도와는 다른 코드를 작성할 수 있습니다.

이러한 문제점을 개선하기 위해 `Assertion`이라는 문구를 사용합니다. Assertion은 컴퓨터 언어로 이루어지며, 컴파일때 영향을 받고, 코드를 이해하는데  도움을 많이 줍니다. 게다가 빌드시 Assertion은 기본적으로 빌드에서 빠지기 때문에 빌드시 성능에도 영향을 미치지 않습니다.

따라서, 주석은 필요할 때 코드를 설명하는것이 아닌 코드를 작성한 의도를 말해주며 제한적으로 사용하고 `Assertion`은 코드에 대한 의도를 명확하게 밝힐 때 자주 사용해야 합니다.

### 절차
1. 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 어서션을 추가한다.

### 예제

예제는 간단하게 제곱근을 예시로 들겠습니다.

```js
//제곱근 계산을 하기 때문에 discountRate는 항상 양수여야 함
if(this.discountRate){
	base = base - (this.discountRate * base);
}
```

```js
assert(this.discountRate >= 0)
if(this.discountRate){
	base = base - (this.discountRate * base);
}
```