---
title: "[Refactoring] 리펙터링 : 첫번째 예시"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-10-14
last_modified_at: 2024-10-14
---

## 리팩터링 서론

이번에 읽을 책은 `리팩터링 2판` 입니다. 리팩터링은 프로그래밍에서 아주 중요한 개념으로 자리잡고 있습니다. 
그렇다면 리팩터링이란 무엇일까요?

> 리팩터링이란, 기능의 수정없이 코드의 구조를 개선하는 작업입니다.

리팩터링을 하면, 서로 엉켜있는 코드를 풀거나, 컴퓨터만 이해할 수 있는 코드를 사람이 이해하기 좋은 코드로 바꿈으로서, 유지보수의 비용을 줄이는 효과를 냅니다. 

하지만 장점만 있는것이 아닙니다. 리팩터링은 말 그대로 잘 작동하는 코드를 수정하는 것이고, 그 과정에서 예상치 못한 버그가 생길 수 있습니다. 어떤 사람들은 이것만 보고 긁어 부스럼 만든다 라고도 하지만, 장담하건데 그 프로젝트는 오래 살아남을 수 없을 것입니다.

이 책의 저자 역시, 리팩터링을 신경쓰며 했던 프로젝트는 롱런을 하고, 그렇지 못한 프로젝트들은 금새 망했다고 합니다. (아마 유저들의 피드백을 반영하려다 보니 유지보수 비용이 높아져서 그런듯 합니다.)

그렇다면, 리팩터링은 코드를 짤 때 마다 해야 하는걸까요? 그렇지는 않습니다. 리팩터링에만 신경쓰다보면 정작 중요한 기능구현이 안될 수 있습니다. 따라서, 리팩터링을 해야하는 시기는 코드를 작성하고 난 뒤에 해야 합니다. 몇몇 사람들은 설계가 완벽하게 되어있으면, 코드짜는 중간에 리팩터링을 하지 않고 코드만 짜면 되는것이 아니냐고 말합니다. 하지만 소프트웨어를 조금이라도 맛본 사람들은 완벽한 설계 따위는 존재하지 않는다는것을 알고 있습니다. 따라서, 코드를 작성하고 리팩터링하는 과정을 꾸준히 하는것이 좋은 설계를 만든다고 볼 수 있습니다.

> 처음부터 완벽한 설계를 갖추기 보다는 개발을 진행하면서 지속적으로 설계한다.

이 과정에서 더 나은 설계가 무엇인지 배울 수 있습니다.

저도 반성을 하게 되는것이, 처음부터 너무 잘할려고 해서 설계에만 집중하다보니 정작 기능구현이 늦어져 프로젝트 기간이 무기한 늘어났던 경험이 있었습니다. 첫술에 배부를 수 없듯이, 꾸준히 하나씩 리팩터링해가며 설계를 개선하고 더 나은 방향을 알아가는것이 필요함을 느꼈습니다.


이 책은 `javascript`라는 언어로 예제가 짜여져 있습니다. 1판은 `java`라서 저한테는 굉장히 친숙한 언어였는데 아쉽게 되었습니다.. 하지만 저자는 `javascript`를 몰라도 리팩터링 과정을 이해하는데는 지장이 없도록 설계하였다고 합니다. `javascript`라는 언어는 잘 모르지만, 리팩터링과정에 요점을 두고 앞으로 포스팅 하도록 하겠습니다.

## 리팩터링 시작해보기

이제 첫 리팩터링을 시작해 봅니다.

다양한 연극을 외주로 받아서 공연하는 극단이 있다고 생각해 봅시다. 공연 요청이 들어오면 연극의 장르와 관객 규모를 기초로 비용을 책정 합니다. 현재 이 극단은 두 가지 장르 (비극, 희극)만 공연 합니다. 그리고, 공연료와 별개로 포인트를 지급해서 다음번 의뢰시 공연료를 할인받을 수도 있습니다.

해당 프로그램을 만들면서 리팩터링하는 과정을 포스팅 하겠습니다.

(해당 함수를 돌리는데 필요한 json 파일은 생략합니다.)

우선, 리팩터링따위는 신경쓰지 않고, 기능 구현에만 초점을 두고 짠 코드 입니다.

```javascript
function statement(invoice, plays) {
    let totalAmount = 0;
    let volumeCredits = 0;
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    const format = new Intl.NumberFor("en-US",
        {
            style: "currency", currency: "USD",
            minimumFractinoDigits: 2
        }).format;

    for (let perf of invoice.performances) {
        const play = plays[perf.playID];
        let thisAmount = 0;

        switch (play.type) {
            case "tragedy": //비극
                thisAmount = 4000;
                if (perf.audience > 30) {
                    thisAmount += 1000 * (perf.audience - 30);
                }
                break;
            case "comedy":
                thisAmount = 3000;
                if (perf.audience > 20) {
                    thisAmount += 10000 + 500 * (perf.audiece - 20);
                }
                thisAmount += 300 * perf.audience;
                break;

            default:
                throw new Error('알 수 없는 장르: ${play.type}');
        }

        //포인트 적립
        volumeCredits += Math.max(perf.audience - 30, 0);
        //희극 관객 5명마다 추가 포인트를 제공
        if ("comedy" == play.type) {
            volumeCredits += Math.floor(perf.audience / 5);
        }

        //청구 내역을 출력함
        result += '${play.name}: ${format(thisAmount/100)} (${perf.audience}석)\ㅜ';

        totalAmount += thisAmount;
    }//for문 종료

    result += '총액: ${format(totalAmount/100)}\n';
    result += '적립 포인트 : ${volumeCredits}점\n';
    return result;
}
```

여기서 switch 문을 주목해 봅니다. switch 문은 현재 한 번의 공연에 대한 요금을 계산하고 있습니다. 이러한 사실은 한눈에 알기 어렵고, 눈이 뚫어져라 코드를 분석해서 얻은 정보 입니다.

나중에 코드를 봐도 까먹지 않으려면, 이 부분의 의도가 무엇인지 명확하게 표현할 필요가 있습니다. 그렇게 하면 다시 분석하지 않아도 코드 스스로가 자신이 하는 일이 무엇인지 이야기해줄 것입니다.

저자는 이러한 방법을 `함수 추출하기`라고 표현 합니다.

## 함수 추출하기

이제 저 한눈에 알아보기 힘든 switch~case 문을 함수로 추출 합니다. 함수를 추출하기 전에, 유효범위를 벗어나는 변수들이 있는지를 확인 합니다.

위의 코드에서 보면 `perf`, `play`, `thisAmount` 가 여기에 속합니다. (`perf`와 `play`인자로 받고, `thisAmount`는 결과값이기에 함수 안에서 지역변수로 선언함)

```javascript
function statement(invoice, plays) {
    let totalAmount = 0;
    let volumeCredits = 0;
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    const format = new Intl.NumberFor("en-US",
        {
            style: "currency", currency: "USD",
            minimumFractinoDigits: 2
        }).format;

    for (let perf of invoice.performances) {
        const play = plays[perf.playID];
        let thisAmount = amountFor(perf, play);//함수 추출하기

        //포인트 적립
        volumeCredits += Math.max(perf.audience - 30, 0);
        //희극 관객 5명마다 추가 포인트를 제공
        if ("comedy" == play.type) {
            volumeCredits += Math.floor(perf.audience / 5);
        }

        //청구 내역을 출력함
        result += '${play.name}: ${format(thisAmount/100)} (${perf.audience}석)\ㅜ';

        totalAmount += thisAmount;
    }//for문 종료

    result += '총액: ${format(totalAmount/100)}\n';
    result += '적립 포인트 : ${volumeCredits}점\n';
    return result;
}

function amountFor(perf, play) {
    let thisAmount = 0;

    switch (play.type) {
        case "tragedy": //비극
            thisAmount = 4000;
            if (perf.audience > 30) {
                thisAmount += 1000 * (perf.audience - 30);
            }
            break;
        case "comedy":
            thisAmount = 3000;
            if (perf.audience > 20) {
                thisAmount += 10000 + 500 * (perf.audiece - 20);
            }
            thisAmount += 300 * perf.audience;
            break;

        default:
            throw new Error('알 수 없는 장르: ${play.type}');
    }

    return thisAmount;//결과값 반환
}
```

함수를 추출하여 switch~case 문이 무엇을 하는 코드인지 눈이 뚫어져라 볼 필요 없이, 함수 이름으로 바로 의도를 파악할 수 있게 되었습니다.

이제 첫 리펙토링을 완료 하였습니다! 저자는 리펙토링을 하고 나서 가장 중요한 단계는 바로 `Test` 하기 라고 합니다. 아무리 간단한 수정이라도 리팩터링 후에는 항상 테스트하는 습관을 들이는것이 바람직 하다고 합니다. 이 이야기를 한줄로 정리하면 다음과 같습니다.

> 조금씩 변경하고 매번 테스트하는 것이 리팩터링 절차의 핵심이다. 프로그램 수정을 작은 단게로 나눠서 진행한다. 그래야 중간에 실수하더라도 버그를 쉽게 찾을 수 있다. 그리고 하나의 리팩터링을 끝낼 때마다 커밋해라.

프로젝트를 하다보면, 리팩토링 해야할것 같은것들을 모아놨다가 한번에 전부 수정하곤 합니다. 이럴때마다, 제 정신적 체력은 많이 닳고, 심지어 자잘한 버그들도 많이 생기게 됩니다. 앞으로는 작은 단위의 기능을 만들어 놓고, 조금씩 변경하는 습관을 들여야겠습니다.

### 변수명 명확히 하기

지금까지 한 작업은, amountFor() 이라는 함수를 중첩함수로 만들어서 statement() 함수에 집어 넣었습니다. 다시 amountFor() 을 보면, 어떤것이 반환값이고, pref 가 뭔지 애매모호한 문제가 있습니다. 다음과 같이 수정합니다.

```javascript
function amountFor(aPerformance, play) {
    let result = 0;

    switch (play.type) {
        case "tragedy": //비극
            thisAmount = 4000;
            if (aPerformance.audience > 30) {
                thisAmount += 1000 * (aPerformance.audience - 30);
            }
            break;
        case "comedy":
            thisAmount = 3000;
            if (aPerformance.audience > 20) {
                thisAmount += 10000 + 500 * (aPerformance.audiece - 20);
            }
            thisAmount += 300 * aPerformance.audience;
            break;

        default:
            throw new Error('알 수 없는 장르: ${play.type}');
    }
    return result;//결과값 반환
}
```

perf 라는 매개변수 이름을 `aPerformance`로 바꾸었습니다. 매개변수의 이름을 더 명확히 함으로써 이 매개변수가 어떤 매개변수인지 한눈에 알아볼 수 있도록 하였습니다.(js 에서는 타입이 없어서 헝가리안표기법을 사용하는데, 지금처럼 매개변수의 역할이 뚜렷하지 않을 때는 부정관사(a/an)을 붙인다고 합니다.)

thisAmount 라는 변수를 `result`로 바꾸었습니다. 함수의 반환값을 `result`로 선언 함으로써 이 함수는 `result`라는 값을 반환한다는 것을 쉽게 알 수 있습니다.

여기서 한가지, 저자가 강조하는 문구가 있습니다.

> 컴퓨터가 이해하는 코드는 바보도 작성할 수 있다. 사람이 이해하도록 작성하는 프로그래머가 진정한 실력자 이다.

코드의 명확성, 가독성을 높이기 위해서 이름을 바꾸는 행동에는 조금의 망설임도 없어야 합니다. 

제가 프로그래밍을 처음 접했을 때, 코드를 작성하면 꼭 줄임말을 쓰곤 했습니다. 코드에 줄임말 (예를 들어, velocity 라는 변수를 v로 줄이는 행위)  을 쓰는것이 멋있다고 생각했기 때문입니다. 그렇게 학원을 수료하고 나서, 시간이 지나고 제 코드를 펼쳐보니 변수이름만 봐서는 이 변수가 어떤 일을 수행하는지를 알기 힘들었습니다. 제가 짠 코드를 제가 이해 못하는것을 느끼고, 변수/함수 의 이름이 엄청 길어지더라도 다음에 봐도 어떤 역할을 하는 변수/함수 인지 알 수 있도록 적는 습관을 길렀습니다.

## 임시 변수를 질의 함수로 바꾸기 및 변수 인라인 하기

여기서 한가지 더, 위의 함수에서 매개변수로 들어오는 `play`라는 값에 주목해 봅시다. `aPerformance` 매개변수는 `for`문을 돌면서 값이 달라지기 때문에, 매개변수로 놓는것이 적절해 보입니다. 하지만 `play`의 경우에는 `aPerformance`의 매개변수에 따라서 값이 달라지기 때문에, `amountFor()` 함수에서 다시 계산하면 됩니다.

그래서, 다음과 같은 함수를 새로 만듭니다.

```javascript
function playFor(aPerformance){
	return plays[aPerformance.playID];
}
```

함수를 새로 만든 뒤, 위의 함수에서 굳이 변수로 선언해주고 있는 play 를 없애 봅시다.

```js
function amountFor(aPerformance) {
    let result = 0;

    switch (playFor(aPerformance).type) {
        case "tragedy": //비극
            thisAmount = 4000;
            if (aPerformance.audience > 30) {
                thisAmount += 1000 * (aPerformance.audience - 30);
            }
            break;
        case "comedy":
            thisAmount = 3000;
            if (aPerformance.audience > 20) {
                thisAmount += 10000 + 500 * (aPerformance.audiece - 20);
            }
            thisAmount += 300 * aPerformance.audience;
            break;

        default:
            throw new Error('알 수 없는 장르: ${playFor(aPerformance).type}');
    }

    return result;//결과값 반환
}
```

위와같이 리팩터링 할 수 있습니다.

하지만 여기서 하나의 의문이 생깁니다. 리팩터링 하기 전에는, `player`라는 변수를 활용해서, 계산을 한번만 하고, 변수에 저장된 값을 돌려썼다면, 리팩터링 후에는 `playFor()` 함수를 만날 때 마다 계산을 해야 해서 성능적인 손실이 생깁니다. 추후에 책에서 리팩터링과 성능에 관해서 이야기를 한다고 합니다. 일단은 이렇게 설명 합니다.

> 설사 리팩터링 후에 심각하게 느려지더라도, 제대로 리팩터링된 코드베이스는 그렇지 않은 코드보다 성능을 개선하기가 훨씬 수월하다.

이렇게까지 해서 지역변수를 없애는 이유는, 추출 작업에서의 이점 때문입니다. 지역변수를 많이 사용하면, 유효범위를 신경써야하기 때문에, 인자를 늘리던지의 작업을 해줘야 합니다. 그래서 저자는 추출작업시에는 지역변수를 없애는 식으로 작업을 진행한다고 합니다.

마찬가지로, 아까 `함수 추출하기`를 통해 thisAmount를 계산하는 함수를 만들었는데, 지금 `thisAmount` 라는 지역변수가 여전히 사용되고 있는것을 볼 수 있습니다. 이것 역시 `변수 인라인하기`를 사용해서 제거 해줍니다.

```js
function statement(invoice, plays) {
    let totalAmount = 0;
    let volumeCredits = 0;
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    const format = new Intl.NumberFor("en-US",
        {
            style: "currency", currency: "USD",
            minimumFractinoDigits: 2
        }).format;

    for (let perf of invoice.performances) {
        //const play = plays[perf.playID]; // -> 변수 인라인하기
        //let thisAmount = amountFor(perf);//함수 추출하기

        //포인트 적립
        volumeCredits += Math.max(perf.audience - 30, 0);
        //희극 관객 5명마다 추가 포인트를 제공
        if ("comedy" == platFor(perf).type) // -> 변수 인라인 하기{
            volumeCredits += Math.floor(perf.audience / 5);

        //청구 내역을 출력함
        //변수 인라인 하기
        result += '${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience}석)\n';

        totalAmount += amountFor(perf);
    }//for문 종료

    result += '총액: ${format(totalAmount/100)}\n';
    result += '적립 포인트 : ${volumeCredits}점\n';
    return result;
}
```