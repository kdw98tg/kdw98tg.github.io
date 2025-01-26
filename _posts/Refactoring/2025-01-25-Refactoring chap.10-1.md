---
title: "[Refactoring] 리팩터링 2판 Chapter.10 part 1 (조건부 로직 간소화)"

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

이번 챕터는 반복문에 관한 내용입니다.

##  조건문 분해하기

조건문은 소프트웨어에서 가장 기초적이고 필수적인 문법입니다. 하지만 그와 동시에 복잡해진다면 프로그램을 이해하는것을 어렵게하는 원흉에 속합니다. 다양한 조건, 그에 따라 동작도 다양한 코드를 작성하면 순식간에 꽤 긴 함수가 탄생합니다.

이 문제를 해결하는 방법으로는, 거대한 코드 블록이 주어지면 코드를 부위별로 분해한 다음 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수 호출로 바꿔줘야 합니다.

### 절차
1. 조건식과 그 조건식에 딸린 조건절을 각각 함수로 추출한다.

### 예제

여름철이면 할인율이 달라지는 어떤 서비스의 요금을 계산한다고 생각합니다. 코드를 바로 짜면 다음과 같이 됩니다.

```js
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd)) {
    charge = quantity * plan.summerRate;
} else {
    charge = quantity * plan.regularRate + plan.regularServiceCharge;
}
```

이 코드를 보면, 조건이 어떤것인지를 확인하려면 코드를 분석해야 하고 많은 시간이 걸립니다. 해당 조건식을 함수로 추출하여 함수명으로 의도를 분명히 해줍니다.

```js
if (isSummer(date)) {
    charge = quantity * plan.summerRate;
} else {
    charge = quantity * plan.regularRate + plan.regularServiceCharge;
}

function isSummer(date) {
    return !date.isBefore(plan.summerStart) && !date.isAfter(plan.summerEnd);
}
```

이렇게 하여 조건을 훨씬 쉽게 알아볼 수 있게 되었습니다. 이제 눈에 거슬리는 부분은, `charge`를 계산하는 부분입니다. 해당 계산과정도 함수로 추출 해줍니다.

```js
if (isSummer(date)) {
    charge = summerCharge(quantity);
} else {
    charge = regularCharge(quantity);
}

function isSummer(date) {
    return !date.isBefore(plan.summerStart) && !date.isAfter(plan.summerEnd);
}

function summerCharge(quantity) {
    return quantity * plan.summerRate;
}

function regularCharge(quantity) {
    return quantity * plan.regularRate + plan.regularServiceCharge;
}
```

위와 같이 리팩터링 함으로서, 코드는 길어졌지만 조건을 훨씬 쉽게 이해할 수 있도록 변경되었습니다.

## 조건식 통합하기

프로그램을 짜다보면, 비교하는 조건은 다르지만 그 결과로 수행하는 동작은 똑같은 코드들이 더러 있습니다. 어차피 같은일을 할거라면 조건 검사도 하나로 통합하는것이 낫습니다. `||` 또는 `&&` 를 사용하여 하나로 합치는 것입니다. 그렇게 한 후, 하나의 맥락이라면 함수로 추출하여 조건식을 간소화 시킵니다.

하지만, 같은 맥락의 조건만 합쳐야지 무분별하게 같은 결과를 내는 조건이라고 전부 합치면 안됩니다.

### 절차
1. 해당 조건식들 모두에 부수효과가 없는지 확인한다.
2. 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.
3. 테스트한다.
4. 조건이 하나만 남을때까지 2~3 과정을 반복한다.
5. 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.

### 예제

다음과 같은 조건문들이 있다고 가정합니다.

```js
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```

위 조건은 하나의 맥락 `이 사람이 장애를 가지고 있는가?`에 대한 내용이고 그에따른 행동도 같습니다.

```js
if (anEmployee.seniority < 2 || anEmployee.monthsDisabled > 12 || anEmployee.isPartTime) return 0;
```

그래서 세가지의 조건을 한군데 모았습니다. 한군데 모으고 나니 조건문 코드가 길어져서 한번에 이해하기가 힘들어졌습니다. 이제, 이 로직을 함수로 추출하여 의도를 분명하게 만듭니다.

```js
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
    return anEmployee.seniority < 2 || anEmployee.monthsDisabled > 12 || anEmployee.isPartTime;
}
```

## 중첩 조건문을 보호 구문으로 바꾸기

이 리팩터링은 갓 프로그래밍을 배운 개발자들에게서 많이 나타나는 문제를 해결해줄 수 있는 방식입니다. 조건문을 갓 배우고 코드를 짤 때, 생각하는대로 짜면 if문과 for문이 중첩되어서 나타나는것을 볼 수 있습니다.

if문 또는 for문이 중첩될 수록, 의도를 파악하기 힘들어지기 때문에 해당 코드를 의도가 분명해지도록 만들어줘야 합니다. 조건에 맞지 않으면 바로 `return`해버리는 식으로 말이죠.

### 절차

1. 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
2. 테스트한다.
3. 1~2 과정을 필요한 만큼 반복한다.
4. 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합한다.

### 예제

피고용인들의 임금을 계산하는 로직 입니다. 우선, 절차대로 조건문을 사용하였습니다. 

```js
function payAmount(employee) {
    let result;
    if (employee.isSeparated) {
        result = { amount: 0, reasonCode: "SEP" };
    }
    else {
        if (employee.isRetired) {
            result = { amount: 0, reasonCode: "RET" };
        }
        else {
            lorem.ipsum(dolor.sitAmet);
            consectetur.adipiscing.elit();
            sed.do.eiusmod = tempor.incididunt.ut.labore.et.dolore.magna.aliqua();
            ut.enim.ad(minim.veniam);
            result = someFinalComputation();
        }
    }
    return result;
}

```

위 코드를 보니, 코드의 Depth가 깊고, 조건을 분석하려면 시간이 많이 걸리게 됩니다. 해당 코드를 보호 구문으로 바꿔서 표현 합니다. 위 코드는 크게 `isSeparated`와 `isRetired`로 분기가 나뉘기 때문에 두 분기를 보호 구문으로 만들어 줍니다. 그리고, 해당 조건을 통과하면, 임금을 계산하는 로직을 수행하도록 합니다.

```js
function payAmount(employee) {
    let result;
    
    if (employee.isSeparated) return { amount: 0, reasonCode: "SEP" };
    if (employee.isRetired) return { amount: 0, reasonCode: "RET" };

    lorem.ipsum(dolor.sitAmet);
    consectetur.adipiscing.elit();
    sed.do.eiusmod = tempor.incididunt.ut.labore.et.dolore.magna.aliqua();
    ut.enim.ad(minim.veniam);
    result = someFinalComputation();
    return result;
}
```

리팩터링된 코드를 보면 훨씬 간단하게 예외에 대해서 이해할 수 있습니다.
