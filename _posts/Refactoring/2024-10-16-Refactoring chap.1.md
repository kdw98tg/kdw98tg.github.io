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

    return result;//결과값 반환
```

perf 라는 매개변수 이름을 `aPerformance`로 바꾸었습니다. 매개변수의 이름을 더 명확히 함으로써 이 매개변수가 어떤 매개변수인지 한눈에 알아볼 수 있도록 하였습니다.(js 에서는 타입이 없어서 헝가리안표기법을 사용하는데, 지금처럼 매개변수의 역할이 뚜렷하지 않을 때는 부정관사(a/an)을 붙인다고 합니다.)

thisAmount 라는 변수를 `result`로 바꾸었습니다. 함수의 반환값을 `result`로 선언 함으로써 이 함수는 `result`라는 값을 반환한다는 것을 쉽게 알 수 있습니다.

여기서 한가지, 저자가 강조하는 문구가 있습니다.

> 컴퓨터가 이해하는 코드는 바보도 작성할 수 있다. 사람이 이해하도록 작성하는 프로그래머가 진정한 실력자 이다.

코드의 명확성, 가독성을 높이기 위해서 이름을 바꾸는 행동에는 조금의 망설임도 없어야 합니다. 

제가 프로그래밍을 처음 접했을 때, 코드를 작성하면 꼭 줄임말을 쓰곤 했습니다. 코드에 줄임말 (예를 들어, velocity 라는 변수를 v로 줄이는 행위)  을 쓰는것이 멋있다고 생각했기 때문입니다. 그렇게 학원을 수료하고 나서, 시간이 지나고 제 코드를 펼쳐보니 변수이름만 봐서는 이 변수가 어떤 일을 수행하는지를 알기 힘들었습니다. 제가 짠 코드를 제가 이해 못하는것을 느끼고, 변수/함수 의 이름이 엄청 길어지더라도 다음에 봐도 어떤 역할을 하는 변수/함수 인지 알 수 있도록 적는 습관을 길렀습니다.




