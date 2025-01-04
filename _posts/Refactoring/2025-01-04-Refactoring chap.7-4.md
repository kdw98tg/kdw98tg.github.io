---
title: "[Refactoring] 리팩터링 2판 Chapter.07 part 4 (캡슐화)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2025-01-04
last_modified_at: 2025-01-04
---

## 중개자 제거하기

이전 포스팅인 part.3 을 보면, 중개자를 둬서 위임을 숨기는 리팩터링을 하였습니다. 이번에 할 것은 그것과 완전히 상반되는 리팩터링 입니다. 중개자를 통해서 수행해야할 역할들이 많아지면, 매번 그럴 때 중개자를 호출하는 메서드만 의미없이 많아집니다. 그럴 때 중개자 제거하기를 통하여 리팩터링 합니다.

그렇다면 언제 중개자를 제거하고, 언제 중개자를 둬야 하는지 헷갈립니다. 역시 답은 없습니다. 6개월전에는 중개자가 필요했는데, 지금보니 중개자를 제거해야 한다면 리팩터링 합니다.

그리고 상황에 따라 중개자 두기 와 중개자 제거하기를 섞어서 써도 됩니다. 자주쓰는 위임은 그대로 놔두고 자주 안쓰는 위임만 제거 합니다. 저자가 강조하는것은 100% 정답은 없고, 상황에 따라서 합리적으로 선택하는것 입니다.

### 절차
1. 위임 객체를 얻는 게터를 만든다.
2. 위임 메서드를 호출하는 클라이언트가 모두 이 게터를 거치도록 수정한다. 하나씩 바꿀 때마다 테스트한다.
3. 모두 수정했다면 위임 메서드를 삭제한다.

### 예제

Person 객체 안에 Department 라는 정보가 있다고 해봅시다. 그 때, 위임을 숨겨서 정보를 반환 한다면 다음과 같이 코딩할 수 있습니다.

```js
class Person {
    constructor(department) {
        this.department = department;
    }

    get manager() {
        return this.department.manager;
    }

    get name() {
        return this.department.name;
    }

    get employees() {
        return this.department.employees;
    }
}

class Department {
    constructor(manager, name, employees) {
        this.manager = manager;
        this.name = name;
        this.employees = employees;
    }

    get manager() {
        return this.manager;
    }

    get employees() {
        return this.employees;
    }

    get departmentName() {
        return this.name;
    }
}
```

이렇게 하면, Department에 대해 숨기면서 Department의 정보를 전부 가져올 수 있습니다. 하지만, 가져올 정보가 많아진다면 문제가 발생할 수 있습니다. 단순히 위임을 하는 메서드가 많아질 수 있습니다. 이때는, 중개자를 지워버립니다.

```js
class Person {
    constructor(department) {
        this.department = department;
    }

    get department() {
        return this.department;
    }
}

class Department {
    constructor(manager, name, employees) {
        this.manager = manager;
        this.name = name;
        this.employees = employees;
    }

    get manager() {
        return this.manager;
    }

    get name() {
        return this.name;
    }

    get employees() {
        return this.employees;
    }
}     
```

위와같이 중개자를 제거합니다. 클라이언트는 이제 Department에 직접 접근하여 정보를 가져오게 됩니다.

```js
const aPerson = new Person(new Department("John", "Sales", ["Alice", "Bob", "Charlie"]));
const manager = aPerson.department.manager;
```

## 알고리즘 교체하기

어려운 문제를 만나면 필히 그 문제를 해결하는 알고리즘도 복잡해지기 마련입니다. 그러다보면, 2중~3중 반복문과 if 문 으로 도배가 되어있는 코드를 만나게 됩니다. 알아보기 힘든 코드는 필히 수정하기도 힘듭니다. 그렇기 때문에, 알고리즘을 간소화 하거나, 교체합니다.

하지만 주의해야 할 점은, 이 작업을 하기전에 메서드를 가능한 잘게 잘 나누었는지 검토해야 합니다. 메서드가 잘게 잘 나눠져있지 않다면, 고치기 쉽지 않을 뿐더러 버그가 날 가능성도 많기 때문입니다.

### 절차
1. 교체할 코드를 함수 하나에 모은다.
2. 이 함수만을 이용해 동작을 검증하는 테스트를 마련한다.
3. 대체할 알고리즘을 준비한다.
4. 정적 검사를 수행한다.
5. 기존 알고리즘과 새 알고리즘의 결과를 비교하는 테스트를 수행한다. 두 결과가 같다면 리팩터링이 끝난다. 그렇지 않다면 기존 알고리즘을 참고해서 새 알고리즘을 테스트하고 디버깅한다.

### 예시

사람의 이름을 찾는 함수를 구현합니다. 현재 코드는 for 문안에 if문이 3개나 박혀있습니다. 

```js
function foundPerson(people) {
    for (let i = 0; i < people.length; i++) {
        if (people[i] === "Don") {
            return "Don";
        }
        if (people[i] === "John") {
            return "John";
        }
        if (people[i] === "Kent") {
            return "Kent";
        }
    }

    return "";
}
```

해당 코드는 잘 작동할지는 몰라도, 새로운 이름이 추가되거나 하면 if문을 또 달아주거나 해야합니다. if문을 주렁주렁 달아놨는데, 사람을 찾는 로직이 달라진다면 어지러워지기 시작하죠.

그래서 C#의 linq 문과 유사한 js에서 제공하는 문법을 활용하여 리팩터링 합니다.

```js
function foundPerson(people) {
    const candidates = ["Don", "John", "Kent"];
    return people.find(person => candidates.includes(person)) || "";
}
```

아직 저기에 있는 배열이 거슬리긴 하지만, 훨씬 유지보수가 쉬운 코드로 바뀌었습니다.

## 7장 마무리

이제 리팩터링의 7장이 마무리가 되었습니다. 7장은 전체적으로 캡슐화에 대한 내용을 강조하며 리팩터링 하였습니다. 이 챕터 역시 엄청 대단한 기법이나 기술보다는 객체지향 프로그래밍을 하는 개발자라면 지켜야 할 원칙에 대해서 기술되어 있었습니다. 이중에서는, 알고 있으면서도 귀찮다는 이유로, 현재 동작하는데 문제가 없다는 이유로 지나쳤던 것들도 몇몇 보였습니다. 리팩터링을 읽고나서, 코드를 짤 때 (특히 라이브러리를 만들 때) 원칙을 지켜가며 하니 훨씬 코드도 간결해지고 눈에 잘 보이게 되었습니다. 객체지향의 원칙을 따른다는것이 이런 이점을 누리기 위해서가 아닐까? 라는 생각을 가지게 되면서 챕터 7을 마무리 하였습니다.