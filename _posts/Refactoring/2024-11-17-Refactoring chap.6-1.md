---
title: "[Refactoring] 리팩터링 2판 Chapter.06 (기본적인 리팩터링)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-11-17
last_modified_at: 2024-11-17
---

이번 챕터에서는 리팩터링의 기법에 대해 설명 합니다. 이 기법들은 `chap.1`에서 간단하게 다루었던 내용 입니다. 해당 내용들을 좀더 깊게 설명해놓은 챕터라고 보면 될것 같습니다.

## 함수 추출하기

함수 추출하기는 가장 많이 사용하는 리팩터링 기법입니다. **코드 조각을 찾아 무슨 일을 하는지 파악한 다음, 독립된 함수로 추출하고 목적에 맞는 이름을 붙입니다.** 이렇게 하면 관리가 힘든 주석을 달 필요 없이 함수 또는 코드라인의 의도를 명확하게 전달할 수 있습니다.

함수 추출하기를 하면 항상 고민되는 녀석이 있습니다. **함수를 나누는 기준을 어떻게 잡을까?** 입니다. 길이에 따라, 재사용성 등등에 따라 나눌 수 있습니다. 물론 길이, 재사용성 둘 다 함수를 나누는데에 중요한 기준이 되지만 저자는 `목적과 구현을 분리`하는 기준을 새로 제시 합니다. 이 기준은 **코드를 보고, 무슨 일을 하는지 파악하는 데 한참이 걸린다면, 그 부분을 함수로 추출한 뒤, 무슨일에 걸맞는지 생각하고 이름을 짓는다**  입니다. 이렇게 하면 10년뒤에 다시 와서 코드를 본다고 해도, 해당 코드가 어떤일을 하는지 다시 생각해볼 필요 없이 함수 이름만 보고도 의도를 파악할 수 있기 때문입니다.

저자는 위 원칙을 적용한 뒤로는 함수를 아주 짧게, 대체로 단 몇줄만 담도록 작성하는 습관이 생겼다고 합니다. 함수를 너무 많이 나누면 나중에 알아보기 힘들지 않을까? 라는 제 고민을 말끔하게 없애주는 문장이었습니다.

또한, 새로 알게된 놀라운 사실은 함수를 많이 호출하면 성능이 느려지지 않을까? 라는 점이었는데, 오히려 짧은 함수들은 컴파일러가 캐싱하기 쉬워져 이점이 생긴다고 합니다.

그리고 주석으로 달린 아래에 명언을 가져와봤습니다. 어디선가 들은 말인데 여기서도 보니 반가웠습니다.

> 최적화를 할 때는 다음 두 규칙을 따른다. 1원칙: 하지마라, 2원칙: 아직 하지마라 - M. A. 잭슨

저번에 포스팅했던 글귀에서 90%는 쓸데없는 최적화 라고 했던 말과 일맥상통 하는 이야기 인것 같습니다. 하나의 기능을 만들고 최적화 하고, 또 만들고 최적화 하는 방식은 나중에 벌어질 일을 다 알고 있다는 듯한 아주 건방진 생각 입니다. 완성을 한 뒤에, 최적화가 필요한 곳이 생겼을 때, 비로서야 최적화를 하는것이 아주 옳은 일임을 알게 되었습니다.

### 함수 추출하기 절차

1. 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다. ("어떻게" 가 아닌 "무엇을" 하는지가 드러나야 한다.)
2. 추출할 코드를 원본 함수에서 복사하여 새 함수에 붙여 넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.
4. 변수를 다 처리했다면 컴파일 한다.
5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다.(즉, 추출한 함수로 일을 위임한다.)
6. 테스트 한다.
7. 다른 코드에 방금 추출한 것과 비슷한 코드가 없는지 살핀다. 있다면 방금 추출한 새 함수를 호출하도록 바꿀지 검토한다.(인라인 코드를 함수 호출로 바꾸기)

### 예시

chap.1 에서 다루었던 내용이므로, 간략하게 작성합니다.

```javascript
//유효범위를 벗어나는 변수가 없을 때
function printOwing(invoice) {
    let outstanding = 0;

    console.log("***********************");
    console.log("**** Customer Owes ****");
    console.log("***********************");

    for (const o of invoice.orders) {
        outstanding += o.amount;
    }

    //마감일(due date)을 기록한다.
    const today = new Date();
    invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);

    //출력
    console.log(`name: ${invoice.customer}`);
    console.log(`amount: ${outstanding}`);
    console.log(`due: ${invoice.dueDate.toLocaleDateString()}`);
}
```

우선, 로그를 찍는 부분이 어떤 역할을 하는 녀석들인지 명확하게 하기 위해 함수로 추출 합니다.

```javascript
//유효범위를 벗어나는 변수가 없을 때

function printOwing(invoice) {
    let outstanding = 0;

    printBanner();

    for (const o of invoice.orders) {
        outstanding += o.amount;
    }

    //마감일(due date)을 기록한다.
    const today = new Date();
    invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);

    //출력
    printDetails(invoice, outstanding);

    function printDetails(invoice, outstanding) {
        console.log(`고객명: ${invoice.customer}`);
        console.log(`채무액: ${outstanding}`);
        console.log(`마감일: ${invoice.dueDate.toLocaleDateString()}`);
    }

    function printBanner() {
        console.log("***********************");
        console.log("**** 채무 표시 ****");
        console.log("***********************");
    }
}

```

다음으로는, 마감일을 설정하는 로직을 함수로 추출 합니다.

```javascript
//유효범위를 벗어나는 변수가 없을 때

function printOwing(invoice) {
    let outstanding = 0;

    printBanner();

    for (const o of invoice.orders) {
        outstanding += o.amount;
    }

    recordDueDate(invoice);

    //출력
    printDetails(invoice, outstanding);

    function printDetails(invoice, outstanding) {
        console.log(`고객명: ${invoice.customer}`);
        console.log(`채무액: ${outstanding}`);
        console.log(`마감일: ${invoice.dueDate.toLocaleDateString()}`);
    }

    function recordDueDate(invoice) {
        const today = new Date();
        invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);
    }

    function printBanner() {
        console.log("***********************");
        console.log("**** 채무 표시 ****");
        console.log("***********************");
    }
}
```


다음은 `outstanding`변수를 추출 합니다. 또한, 계산된 `outstanding`변수는 불변값이므로 `const`로 바꿔 줍니다.

```javascript
z//유효범위를 벗어나는 변수가 없을 때

function printOwing(invoice) {

    printBanner();
    const outstanding = calculateOutstanding(invoice);
    recordDueDate(invoice);
    printDetails(invoice, outstanding);


    function printDetails(invoice, outstanding) {
        console.log(`고객명: ${invoice.customer}`);
        console.log(`채무액: ${outstanding}`);
        console.log(`마감일: ${invoice.dueDate.toLocaleDateString()}`);
    }

    function calculateOutstanding(invoice) {
        let result = 0;
        for (const o of invoice.orders) {
            result += o.amount;
        }
        return result;
    }

    function recordDueDate(invoice) {
        const today = new Date();
        invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);
    }

    function printBanner() {
        console.log("***********************");
        console.log("**** 채무 표시 ****");
        console.log("***********************");
    }
}

```


## 함수 인라인하기

함수 추출하기를 너무 과다하게 써버리면, 쓸데없이 잘게 나눠진 함수들이 존재합니다. 혹여나 그런것들을 발견했다면, 다시 함수로 인라인 합니다. 잘못 추출된 함수들을 원래 함수로 합친 다음, 필요하면 원하는 형태로 다시 추출 합니다.

함수가 너무 잘게 쪼개져 간접 호출을 너무 과하게 쓰는 코드도 흔한 인라인 대상 입니다. `함수 인라인하기`를 활용하면 유용한 함수들만 남기고 나머지는 제거할 수 있습니다.

### 절차

1. 다형 메서드인지 확인한다.(서브클래스에서 오버라이드하는 메서드는 인라인 하면 안됨)
2. 인라인할 함수 본문으로 교체한다.
3. 각 호출문을 함수 본문으로 교체한다.
4. 하나씩 교체할 때마다 테스트 한다.
5. 함수 정의를 삭제한다.

### 예시

가장 간단한 경우를 먼저 살펴 봅니다.

```javascript
function rating(driver) {
    return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
    return driver.numberOfLateDeliveries > 5;
}
```

해당 코드는 `rating`을 가져오는 함수 입니다. 여기서는 너무 잘게 쪼갠 나머지 간단한 조건문에도 함수를 하나 더 나눠서 간접호출을 하나 더 늘린 모습입니다.

해당 코드를 인라인 하여 하나의 함수로 만들어 줍니다.

```javascript
// 변경 후
function getRating(driver) {
    return driver.numberOfLateDeliveries > 5 ? 2 : 1;
}
```


다음은 조금 더 일이 많은 코드를 예시로 들어 봅니다.

```javascript
function reportLines(customer) {
    const lines = [];
    gatherCustomerData(lines, customer);
    return lines;
}

function gatherCustomerData(lines, customer) {
    lines.push(["name", customer.name]);
    lines.push(["location", customer.location]);
}
```

해당 코드는 아까 봤던 예제처럼 간단히 붙여넣기로는 인라인이 불가능합니다. 하지만, 배열에 값을 집어넣는 과정을 쓸데없이 함수로 나눠놨기 때문에, 해당 코드를 인라인 해줍니다.

```javascript
function reportLines(customer) {
    const lines = [];
    lines.push(["name", customer.name]);
    lines.push(["location", customer.location]);
    return lines;
}
```
