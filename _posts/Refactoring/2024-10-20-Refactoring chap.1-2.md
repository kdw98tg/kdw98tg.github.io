---
title: "[Refactoring] 리팩터링 2판 Chapter.01 (반복문 쪼개기, 문장 슬라이드하기)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-10-20
last_modified_at: 2024-10-20
---

## 함수 추출하기, 변수 인라인 하기

이전의 포스팅을 통해 리펙토링된 코드를 보면 다음과 같습니다.

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
        result += '${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience}석)\n';

        totalAmount += amountFor(perf);
    }//for문 종료

    result += '총액: ${format(totalAmount/100)}\n';
    result += '적립 포인트 : ${volumeCredits}점\n';
    return result;
}

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

function playFor(aPerformance) {
    return plays[aPerformance.playID];
}
```

저번 포스팅에서는 함수 추출하기, 변수 인라인하기를 통해서 코드를 리팩터링 했습니다. 이번 포스팅에도 이어서 변수 인라인하기를 적용 합니다.

현재 다음의 코드가 눈에 거슬립니다.

```js
volumeCredits += Math.max(perf.audience - 30, 0);
        //희극 관객 5명마다 추가 포인트를 제공
        if ("comedy" == platFor(perf).type) // -> 변수 인라인 하기{
            volumeCredits += Math.floor(perf.audience / 5);

```

해당 코드도 한번에 의도를 파악하기가 힘듭니다. 그래서 `함수 추출하기`를 통해서 새로운 함수를 만들어 의도를 명확히 합니다.

```js
function volumeCreditsFor(aPerformance){
    let volumeCredits = 0;

    volumeCredits += Math.max(aPerformance.audience - 30, 0);
    if ("comedy" == playFor(aPerformance).type) // -> 변수 인라인 하기{
        volumeCredits += Math.floor(aPerformance.audience / 5);

    return volumeCredits;
}
```

다음은 마찬가지로, `format`변수를 주목 합니다. 해당 코드가 어떤일을 하는지 `함수 추출하기`를 통해서 명확히 합니다. 또한, `변수 인라인하기`를 통해서 지역변수 하나를 더 줄입시다. 해당 함수를 추출하였을 때, 이름을 생각해봐야 하는데 단순히 `format`으로 하면 이 함수의 의도를 충분히 설명할 수 없습니다. 또한, 코드 중간에 들어가는 함수라서 너무 긴 이름은 부적절할 수 있습니다. 

해당 함수는 화폐 단위를 맞춰주는 역할이기 때문에 간단하게 `usd`로 명명 합니다.

```js
function usd(aNumber) {
    return new Intl.NumberFor("en-US",
        {
            style: "currency", currency: "USD",
            minimumFractinoDigits: 2
        }).format;
}
```

이처럼 함수의 이름을 지을때, 굉장히 많은 생각을 하게 되는것 같습니다. 해당 함수의 이름을 쭉 풀어쓰기에는 함수의 이름이 너무 길어져 가독성을 해칠 수 있지만, 그렇다고 너무 짧게되면 나중에 봤을 때 어떤일을 하는 함수인지 알기 힘듭니다. (변수명 짓기, 함수명 짓기 등을 도와주는 사이트도 있음)

저자는 처음에는 당장 떠오르는 최선의 이름을 사용하다가, 나중에 더 좋은 이름이 떠오를 때 바꾸는 식이 좋다고 합니다. 이름짓기가 중요하다고는 하지만, 주객이 전도되어 이름짓는데만 너무 열정을 쏟으면 안된다 라는 말을 하는것 같았습니다.

## 반복문 쪼개기, 문장 슬라이드 하기

위에서 `volumeCreditFor()`이라는 함수를 추출하였습니다. 함수를 추출하여 의도를 명확히 한것 까지는 좋으나, 여전히 지역변수에 담아놓고 사용을 합니다. 또한, `volumeCreditFor()`은 값을 누적하기 때문에, 나중에 리팩터링하기 까다로울 수 있습니다.

우선, `반복문 쪼개기`를 통해서 별도의 반복문에서 값을 누적하도록 만듭니다,.

```js
function statement(invoice, plays) {
    let totalAmount = 0;
    let volumeCredits = 0;
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    for (let perf of invoice.performances) {
        //const play = plays[perf.playID]; // -> 변수 인라인하기
        //let thisAmount = amountFor(perf);//함수 추출하기

        //청구 내역을 출력함
        result += '${playFor(perf).name}: ${usd(amountFor(perf)/100)} (${perf.audience}석)\n';
        totalAmount += amountFor(perf);
    }//for문 종료

    for (let perf of invoice.performances) {
        //포인트 적립
        volumeCredits += volumeCreditsFor(perf);
    }

    result += '총액: ${usd(totalAmount/100)}\n';
    result += '적립 포인트 : ${volumeCredits}점\n';
    return result;
}
```

그 다음으로는 `문장 슬라이드하기`를 적용 합니다. 이 과정은 단순히 가독성을 높이기 위함 입니다. 위쪽에서 선언된 `let volumeCredits = 0` 문장을 `volumeCredits`를 계산하는 쪽으로 이동 시킵니다.

```js
function statement(invoice, plays) {
    let totalAmount = 0;
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    for (let perf of invoice.performances) {
        //const play = plays[perf.playID]; // -> 변수 인라인하기
        //let thisAmount = amountFor(perf);//함수 추출하기

        //청구 내역을 출력함
        result += '${playFor(perf).name}: ${usd(amountFor(perf)/100)} (${perf.audience}석)\n';
        totalAmount += amountFor(perf);
    }//for문 종료

    let volumeCredits = 0;// -> 문장 슬라이드 하기
    for (let perf of invoice.performances) {
        //포인트 적립
        volumeCredits += volumeCreditsFor(perf);
    }

    result += '총액: ${usd(totalAmount/100)}\n';
    result += '적립 포인트 : ${volumeCredits}점\n';
    return result;
}
```

이렇게 하는 이유에는 가독성의 측면도 있지만, 나중에 해당 기능을 추출할 때 값 갱신과 관련된 코드들을 한데 모아두면 해당 기능을 추출하기가 수월해진다는 장점이 있습니다.

우선, `함수 추출하기`를 사용하여 함수로 추출부터 합니다.

```js
function totalVolumeCredits() {
    let volumeCredits = 0;
    for (let perf of invoice.performaces) {
        volumeCredits += volumeCreditsFor(perf);
    }
    return volumeCredits;
}

function statement(invoice, plays) {
    let totalAmount = 0;
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    for (let perf of invoice.performances) {
        //청구 내역을 출력함
        result += '${playFor(perf).name}: ${usd(amountFor(perf)/100)} (${perf.audience}석)\n';
        totalAmount += amountFor(perf);
    }//for문 종료

    let volumeCredits = totalVolumeCredits(perf);// -> 함수 추출하기

    result += '총액: ${usd(totalAmount/100)}\n';
    result += '적립 포인트 : ${volumeCredits}점\n';
    return result;
}
```

그다음에는 굳이 필요 없는 지역변수를 `변수 인라인하기`를 통해 없애 줍니다.

```js
function statement(invoice, plays) {
    let totalAmount = 0;
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    for (let perf of invoice.performances) {
        //청구 내역을 출력함
        result += '${playFor(perf).name}: ${usd(amountFor(perf)/100)} (${perf.audience}석)\n';
        totalAmount += amountFor(perf);
    }//for문 종료

    //let volumeCredits = totalVolumeCredits(perf);

    result += '총액: ${usd(totalAmount/100)}\n';
    result += '적립 포인트 : ${totalVolumeCredits(perf)}점\n';//-> 변수 인라인 하기
    return result;
}
```

또, 눈에 거슬리는 부분이 있습니다. 바로 이 부분 입니다.

```js
    for (let perf of invoice.performances) {
        //청구 내역을 출력함
        result += '${playFor(perf).name}: ${usd(amountFor(perf)/100)} (${perf.audience}석)\n';
        totalAmount += amountFor(perf);
    }//for문 종료
```

해당부분 역시 값을 중첩을 하고 있습니다. 이것역시 `반복문 쪼개기`를 통해 리팩터링 해줍니다.

```js
function totalAmount() {
    let totalAmount = 0;
    for (let perf of invoice.performances) {
        totalAmount += amountFor(perf);
    }
    return totalAmount;
}

function statement(invoice, plays) {
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    for (let perf of invoice.performances) {
        //청구 내역을 출력함
        result += '${playFor(perf).name}: ${usd(amountFor(perf)/100)} (${perf.audience}석)\n';
    }//for문 종료

    let totalAmount = totalAmount();//->반복문 쪼개기

    result += '총액: ${usd(totalAmount/100)}\n';
    result += '적립 포인트 : ${totalVolumeCredits(perf)}점\n';//-> 변수 인라인 하기
    return result;
}
```

마지막으로 지역변수의 이름들을 저자의 코드 스타일로 바꿔줍니다.(일단 따라합니다.)

```js
function totalVolumeCredits() {
    let result = 0;
    for (let perf of invoice.performaces) {
        result += volumeCreditsFor(perf);
    }
    return result;
}

function totalAmount() {
    let result = 0;
    for (let perf of invoice.performances) {
        result += amountFor(perf);
    }
    return result;
}
```

그리고 함수들을 모두 인라인 함수로 바꿔줍니다.(인라인 함수는 함수 안에 있는 함수입니다. 저는 써본적이 없지만, js 에서는 이렇게 쓰나봅니다. 일단 따라 합니다.)

그래서, 완성된 코드를 올리면 다음과 같습니다.

```js
function statement(invoice, plays) {
    let result = '청구 내역 (고객명: ${invoice.customer})\n'

    for (let perf of invoice.performances) {
        //청구 내역을 출력함
        result += '${playFor(perf).name}: ${usd(amountFor(perf)/100)} (${perf.audience}석)\n';
    }//for문 종료

    let totalAmount = totalAmount();//->반복문 쪼개기

    result += '총액: ${usd(totalAmount/100)}\n';
    result += '적립 포인트 : ${totalVolumeCredits(perf)}점\n';//-> 변수 인라인 하기
    return result;

    function volumeCreditsFor(aPerformance) {
        let volumeCredits = 0;

        volumeCredits += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" == playFor(aPerformance).type) // -> 변수 인라인 하기{
            volumeCredits += Math.floor(aPerformance.audience / 5);

        return volumeCredits;
    }

    function usd(aNumber) {
        return new Intl.NumberFor("en-US",
            {
                style: "currency", currency: "USD",
                minimumFractinoDigits: 2
            }).format;
    }

    function totalVolumeCredits() {
        let result = 0;
        for (let perf of invoice.performaces) {
            result += volumeCreditsFor(perf);
        }
        return result;
    }

    function totalAmount() {
        let result = 0;
        for (let perf of invoice.performances) {
            result += amountFor(perf);
        }
        return result;
    }

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

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }
}
```

일단 코드를 쭉 보면, 기능별로 함수들이 잘 쪼개져 있고, 보기에도 훨씬 좋아졌습니다. 결과적으로 각 계산 과정은 물론, 전체 흐름을 이해하기가 훨씬 쉬워졌습니다.

하지만, 계속 거슬리는 것은 리팩터링으로 코드가 더 읽기 쉬워졌다는 점이지만, 1번이었던 for 문을 함수로 나누고 인라인 하여, 함수를 호출할 때 마다 반복문을 돌려 성능의 저하가 일어날 수 있다는 점입니다.

여기서 저자는 다음과 같이 답합니다.

> 괜찮다. 이정도 중복은 성능에 미치는 영향이 미미할 때가 많다. 실제로 리팩터링 전과 후의 속도 차이를 측정하면 거의 차이를 느끼지 못한다. 또한, 똑똑해진 컴파일러들은 최신 캐싱 기법 등으로 무장하고 있어서 우리의 직관을 초월하는 결과를 내어준다. 하지만, 대부분 그렇다는 것이지 반드시 그렇다는것은 아니다. 
> 
> 성능이 느려질때도 있지만, 그래도 나는 개의치않고 리팩터링 한다. 잘 리팩터링된 코드는 성능개선도 더 쉽기 때문이다. 그래서 한마디로 요약하자면 다음과 같다.
> 
> 성능저하가 특별한 경우가 아니라면 일단 무시해라. 성능저하가 심하다면, 리팩터링 하고 나서 성능을 개선하자.

저번에 읽었던 포스팅에서는 다음과 같은 말을 들었습니다. 컴퓨터는 생각보다 빠르다 라는 말이었습니다. 물론 성능을 개선하고 최적화를 하는것은 매우 중요한 일입니다. 하지만, 그 과정에서 사람이 이해하기 힘들어지는 코드가 된다면, 그 소프트웨어는 개선하기가 매우 어려워 질것입니다. 사실 아직 좀 꺼림직하긴 하지만, 일단 저자가 하라는대로 리팩터링을 먼저하는 습관을 길러야겠습니다.

마지막으로, 저자는 다음과같은 말은 합니다. 

> 우리는 지금 반복문 쪼개기 -> 문장 슬라이드하기 -> 함수 추출하기 -> 변수 인라인하기 를 통해 순차적으로  리팩터링을 했다.
> 
> 이정도 코드는 복잡하지 않기 때문에 굳이 순서를 나누고 하지 않아도 큰 지장이 없다. 하지만, 코드라인이 많아지고 로직이 복잡해지면 단계를 더 작게 나누는 일을 가장 먼저 한다.

위 말에서 크게 공감되었던것이 있습니다. 프로그래밍 강의를 하면 학생들에게 항상 하는 말이 있습니다. '`프로그래밍이란 사람이 할 수 있는 일을 컴퓨터에게 시키는 작업이다`. 따라서 사람이 생각하고 효율적으로 일을 할 수 있는 방식을 구현하는것이 (물론 조금은 차이가 있을 수 있습니다) 중요하다. 그렇게 하기 위해서는 행동 하나하나에 절차지향적인 생각을 할 수 있도록 훈련하는것이 중요하다.' 입니다.

리팩터링도 마찬가지로, 복잡한 단계에 마주하면 책에서 배웠던 기술들을 활용해 작은 순서대로 나열해보는것이 중요한것 같습니다.


