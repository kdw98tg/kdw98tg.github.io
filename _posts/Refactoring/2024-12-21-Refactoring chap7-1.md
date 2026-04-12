---
title: "[Refactoring] 리팩터링 2판 Chapter.07 part 1 (캡슐화)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-12-21
last_modified_at: 2024-12-21
---

모듈을 분리하는 가장 중요한 기준은 아마도 시스템에서 각 모듈이 자신을 제외한 다른 부분에 드러내지 않아야 할 비밀을 얼마나 잘 숨기느냐에 있습니다. 객체지향에서 모듈이란 아주 중요한 개념을 가집니다. 모듈을 만드는 가장 기본적인 방법은 클래스 입니다. 이번 챕터에서는 모듈을 만들면서 리팩터링 하는 방법을 설명합니다.

저도 언젠가부터 캡슐화를 사용하여 같은 기능을 하는 녀석들을 모으고, 결합도를 최대한 없애는 식으로 코딩을 하고 있습니다. 그렇게 하다 보니 프레임워크에서 제공하는 클래스보다 제가 만든 사용자 정의 자료형이 훨씬 많아지게 되었습니다. 

이렇게 했을때의 장점은 다음과 같습니다.
- 기능의 확장 용이
- 기능의 은닉화
- 재사용 가능
- 기능 변경시 용이함

지금 생각나는 장점으로는 4가지정도가 있습니다. 아마 더 많은 장점을 가지고 있을겁니다. 그리고 단점은 클래스가 많아져서 관리하기 힘들다 정도밖엔 없는것 같습니다. 이번 챕터에서는 해당 기법들을 어떻게 잘 사용할 것인가를 다룹니다.

## 레코드 캡슐화 하기
가변 데이터를 저장하는 용도로 레코드보다 객체를 사용한다. 객체를 사용하면 어떻게 저장했는지를 숨긴 채 세 가지 값을 각각의 메서드로 제공할 수 있기 때문이다.

반대로 불변 데이터를 저장할 때에는 그냥 레코드를 사용한다.

### 절차
1. 레코드를 담은 변수를 캡슐화 한다.
2. 레코드를 감싼 단순한 클래스로 해당 변수의 내용을 교체한다. 이 클래스에 원본 레코드를 반환하는 접근자도 정의하고, 변수를 캡슐화하는 함수들이 이 접근자를 사용하도록 수정한다.
3. 테스트한다.
4. 원본 레코드를 대신 새로 정의한 클래스 타입의 객체를 반환하는 함수들을 새로 만든다.
5. 레코드를 반환하는 예전 함수를 사용하는 코드를 4번에서 만든 새 함수를 사용하도록 바꾼다. 필드에 접근할 때에는 객체의 접근자를 사용한다. 적절한 접근자가 없다면 추가한다. 한 부분을 바꿀 때마다 테스트한다.
6. 클래스에서 원본 데이터를 반환하는 접근자와 원본 레코드를 반환하는 함수들을 제거한다.
7. 테스트한다.
8. 레코드의 필드도 데이터 구조인 중첩 구조라면 레코드 캡슐화하기와 컬렉션 캡슐화하기를 재귀적으로 적용한다.

### 예제
간단한 레코드를 먼저 캡슐화 해본다.

```javascript
const organiztion={name:"애크미 구스베리", country:"GB"};

result += `<h1>${organiztion.name}</h1>`;
result += `<h2>${organiztion.country}</h2>`;
```

organiztion 이라는 레코드를 하나 만들었다. 그것을 사용하는 방식은 h1, h2 테그 안에 있는 코드와 같습니다.

이제 이것을 클래스로 캡슐화 하여 나타냅니다.

```javascript
class Organization{
    constructor(data){
        this.name = data.name;
        this.country = data.country;
    }
    set name(value){
        this._name = value;
    }
    get name(){
        return this._name;
    }
    get country(){
        return this._country;
    }
    set country(value){
        this._country = value;
    }
}

const organization = new Organization({name:"애크미 구스베리", country:"GB"});

result += `<h1>${organization.name}</h1>`;
result += `<h2>${organization.country}</h2>`;
```

이렇게 하면 입력 데이터 레코드와의 연결을 끊어준다는 이점이 생깁니다. 특히 이 레코드를 참조하여 캡슐화를 깰 우려가 있는 코드가 많을 때 좋습니다. 데이터를 개별 필드로 펼치치 않았다면 data를 대입할 때 복제하는 식으로 합니다.

## 컬렉션 캡슐화 하기
저자는 가변 데이터를 모두 캡슐화 하는 편이라고 합니다. 그렇게 하면 데이터 구조가 언제 어떻게 수정되는지 파악하기 쉬워서 필요한 시점에 데이터 구조를 변경하기도 쉬워지기 때문입니다.

저자는 컬렉션 (List, Queue, Stack 등등)마저도 캡슐화 해서 사용한다고 합니다. 문제는 항상 데이터를 수정할 때 발생합니다. 그래서 getter 에서는 리스트의 원본이 아닌 복사본을 만들어서 넘긴다고 합니다. 그러면 원본 데이터의 무결을 보장할 수 있기 때문입니다. 컬렉션의 크기가 매우 크다면, 성능에 영향을 미칠 수 있긴 하지만, 크기가 매우 큰 컬렉션은 거의 없을 뿐더러, 많다면 무언가 잘못된 것입니다.

### 절차
1. 아직 컬렉션을 캡슐화 하지 않았다면 변수 캡슐화하기 부터 한다.
2. 컬렉션에 원소를 추가/제거하는 함수를 추가한다.
3. 정적 검사를 수행한다.
4. 컬렉션을 참조하는 부분을 모두 찾는다. 컬렉션의 변경자를 호출하는 코드가 모두 앞에서 추가한 추가/제거 함수를 호출하도록 수정한다. 하나씩 수정할 때마다 테스트한다.
5. 컬렉션 게터를 수정해서 원본 내용을 수정할 수 없는 읽기전용 프록시나 복제본을 반환하게 한다.
6. 테스트한다.

### 예제
이번 예제는 Person 이라는 클래스가 Course의 리스트를 가지고 있다고 가정합니다. Person의 컬렉션을 수정하기 위한 함수들을 만들어 캡슐화를 하는 과정입니다.

우선, Person 클래스를 정의합니다.

```javascript
class Person {
    constructor(name) {
        this.name = name;
        this.courses = [];
    }
    get name() {
        return this._name;
    }
    get courses() {
        return this._courses;
    }
    set courses(aList) {
        this._courses = aList;
    }
}
```

현재는 컬렉션의 setter를 열어서 컬렉션을 수정하는 방식 입니다.

다음으로는 Course 클래스를 정의 합니다.

```js
class Course {
    constructor(name, isAdvanced) {
        this.name = name;
        this.isAdvanced = isAdvanced;
    }
    get name() {
        return this._name;
    }
    get isAdvanced() {
        return this._isAdvanced;
    }
}
```

Person 과 Courses 를 정의하고, Person의 컬렉션을 정의하려고 보니 다음과 같이 정의 해야 합니다.

```js
for(const name of ['Kai', 'John', 'Jane', 'Jim']) {
    aPerson.courses.push(new Course(name, true));
}
```

하지만, 이렇게 하면 컬렉션을 수정할 때, Person이 제어권을 잃으므로, 캡슐화가 깨진다고 볼 수 있습니다. 따라서 Person 클래스에 `addCourse`, `removeCourse`메서드를 추가하여 컬렉션을 캡슐화 해줍니다.

```js
class Person {
    constructor(name) {
        this.name = name;
        this.courses = [];
    }
    get name() {
        return this._name;
    }
    get courses() {
        return this._courses;
    }
    set courses(aList) {
        this._courses = aList;
    }

    addCourse(aCourse) {
        this.courses.push(aCourse);
    }
    removeCourse(aCourse) {
        this.courses = this.courses.filter(c => c.name !== aCourse.name);
    }
}
```

Person 클래스의 컬렉션을 캡슐화 하게 되면 다음과 같이 컬렉션을 수정할 수 있습니다.

```js
for(const name of ['Kai', 'John', 'Jane', 'Jim']) {
    aPerson.addCourse(new Course(name, true));
}
```

