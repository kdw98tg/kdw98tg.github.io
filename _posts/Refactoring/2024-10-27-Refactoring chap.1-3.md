---
layout: post

title: "[Refactoring] 리팩터링 2판 Chapter.01 (단계 쪼개기)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-10-27
last_modified_at: 2024-10-27
---

지난 포스팅에서 리펙토링 한 코드 입니다.

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

위 코드를 살펴보면, 리팩터링을 통해 코드의 구조가 한결 나아진 것을 볼 수 있습니다. 원래는 `statement()`라는 함수 하나였지만, 지금은 함수가 늘어나긴 했지만, `statement()`코드는 총 8줄로 구성되어 있습니다. **결론적으로, 각 계산 과정은 물론, 전체 흐름을 이해하기가 훨씬 쉬워졌습니다.**

지금까지의 리팩터링은 코드의 논리적인 요소를 파악하기 쉽도록 하는것이 목적이었습니다. 이번 포스팅에서는 코드의 수행 단계에 따라서 리팩터링을 해봅니다. 저자는 이 과정을 `단계 쪼개기`라고 표현 합니다. (현재 statement() 는 HTML 에 표현할 문자열을 생성하는 역할을 함. 이제부터 statement() 이 문자열을 HTML 에 표현하기 위한 작업을 함.)

우선, `함수 추출하기`를 사용합니다. `statement()`는 현재 텍스트를 출력하는 역할을 하고 있습니다. 이것을 `renderPlainText()`라는 함수로 추출 합니다. 이 과정에서 중첩함수와 `statement()`는 분리하도록 합니다.

```js
//중첩함수 분리 (renderPlainText 에서 계산한 결과값만 반환 받을 것임)
function statement(invoice, plays) {
    return renderPlainText(invoice, plays);
}


function renderPlainText(invoice, plays) {
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

그리고, 두 단계 사이의 중간 데이터 구조 역할을 할 객체를 만들어서 `renderPlainText()` 에 인수로 전달 합니다. 이 과정에서 `renderPlainText()`함수에 필요한 값들을 `statement()`에서 계산하여 전달하는 과정을 수행 합니다.

그러기 위해서 `renderPlainText()`안에 있는 중첩함수들을 다시 `statement()`의 중첩함수로 추출 합니다. 이 과정에서 `enrichPerformance()`라는 함수를 생성하여 중간 데이터를 전달하게 만듭니다. 여기서 저자는 다음과 같이 말합니다.

> 데이터를 전달 할 때, 가변 데이터는 금방 상하기 때문에 최대한 불변 처럼 취급한다.

그래서 `enrichPerformance()`함수는 얕은 복사를 사용하여 구성합니다. 

해당 과정을 거친 코드는 아래와 같습니다.

```js
//중첩함수 분리 (renderPlainText 에서 계산한 결과값만 반환 받을 것임)
function statement(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);//totalAmount 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);//totalVolumeCredits 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
    return renderPlainText(statementData, plays);

    function enrichPerformance(aPerformance) {
        const result = Object.assign({}, aPerformance);//얕은 복사 수행
        result.play = playFor(result);//playFor 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
        result.amount = amountFor(result);//amountFor 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
        result.volumeCredits = volumeCreditsFor(result);//volumeCreditsFor 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
        return result;
    }

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }

    function amountFor(aPerformance) {
        let result = 0;

        switch (aPerformance.play.type) {
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

    function volumeCreditsFor(aPerformance) {
        let volumeCredits = 0;

        volumeCredits += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" == aPerformance.play.type) // -> 변수 인라인 하기{
            volumeCredits += Math.floor(aPerformance.audience / 5);

        return volumeCredits;
    }

    function totalAmount(data) {
        // let result = 0;
        // for (let perf of data.performances) {
        //     result += perf.amount;
        // }
        // return result;
        return data.performances.reduce((total, p) => total + p.amount, 0);
    }

    function totalVolumeCredits(data) {
        // let result = 0;
        // for (let perf of data.performances) {
        //     result += perf.volumeCredits;
        // }
        // return result;

        return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
    }
}

function renderPlainText(data, plays) {
    let result = '청구 내역 (고객명: ${data.customer})\n'

    for (let perf of invoice.performances) {
        //청구 내역을 출력함
        result += '${perf.play.name}: ${usd(amountFor(perf.amount)/100)} (${perf.audience}석)\n';//playFor 함수 대체
    }//for문 종료

    let totalAmount = totalAmount();//->반복문 쪼개기

    result += '총액: ${usd(data.totalAmount/100)}\n';
    result += '적립 포인트 : ${data.totalVolumeCredits}점\n';//-> 변수 인라인 하기
    return result;

    function usd(aNumber) {
        return new Intl.NumberFor("en-US",
            {
                style: "currency", currency: "USD",
                minimumFractinoDigits: 2
            }).format;
    }
}
```

다음으로는, `반복문을 파이프라인으로 바꾸기` 를 적용합니다. 

```js
   function totalAmount(data) {
        // let result = 0;
        // for (let perf of data.performances) {
        //     result += perf.amount;
        // }
        // return result;
        return data.performances.reduce((total, p) => total + p.amount, 0);
    }

    function totalVolumeCredits(data) {
        // let result = 0;
        // for (let perf of data.performances) {
        //     result += perf.volumeCredits;
        // }
        // return result;

        return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
    }
```

C#의 Linq 와 같은 역할을 하는 식으로 나타내어 코드 라인수를 줄이고, 가독성을 향상시키는것 같습니다.

이제, 이렇게 하고 나니, `statement()` 함수가 꽤나 분량이 많아졌습니다. 중첩함수를 분리해서, 값을 계산하는 부분과 분리해 줍니다.

```js
function statement(invoice, plays) {
    return renderPlainText(createStatementData(invoice, plays));
}

function createStatementData(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);//totalAmount 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);//totalVolumeCredits 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성

//...여러 중첩 함수들....
}
```

이제, 별도의 파일로 저장하도록 합니다. `createStatementData.js`를 만들어줍니다. 그리고 분리된 `statement.js` 파일에 html 을 렌더링 할 수 있는 코드도 작성 합니다. (이 과정에서 usd 함수는 html을 렌더링 할 때도 사용할 수 있도록 최상위로 옮김)

🗅 **<span style="color: #c03a92">statement.js</span>**

```js
import createStatementData from './createStatementData.js'

function statement(invoice, plays) {
    return renderPlainText(createStatementData(invoice, plays));
}

function renderPlainText(data, plays) {
    let result = '청구 내역 (고객명: ${data.customer})\n'

    for (let perf of invoice.performances) {
        //청구 내역을 출력함
        result += '${perf.play.name}: ${usd(amountFor(perf.amount)/100)} (${perf.audience}석)\n';//playFor 함수 대체
    }//for문 종료

    let totalAmount = totalAmount();//->반복문 쪼개기

    result += '총액: ${usd(data.totalAmount/100)}\n';
    result += '적립 포인트 : ${data.totalVolumeCredits}점\n';//-> 변수 인라인 하기
    return result;
}

function htmlStatement(invoice, plays) {
    return renderHtml(createStatementData(invoice, plays));
}

function renderHtml(data) {
    let result = `<h1>Statement for ${data.customer}</h1>\n`;
    result += "<table>\n";
    result += "<tr><th>play</th><th>seats</th><th>cost</th></tr>";
    for (let perf of data.performances) {
        result += `  <tr><td>${perf.play.name}</td><td>${perf.audience}</td>`;
        result += `<td>${usd(perf.amount)}</td></tr>\n`;
    }
    result += "</table>\n";
    result += `<p>Amount owed is <em>${usd(data.totalAmount)}</em></p>\n`;
    result += `<p>You earned <em>${data.totalVolumeCredits}</em> credits</p>\n`;
    return result;
}

function usd(aNumber) {
    return new Intl.NumberFor("en-US",
        {
            style: "currency", currency: "USD",
            minimumFractinoDigits: 2
        }).format;
}
```

🗅 **<span style="color: #c03a92">createStatementData.js</span>**

```js
function createStatementData(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);//totalAmount 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);//totalVolumeCredits 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성


    function enrichPerformance(aPerformance) {
        const result = Object.assign({}, aPerformance);//얕은 복사 수행
        result.play = playFor(result);//playFor 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
        result.amount = amountFor(result);//amountFor 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
        result.volumeCredits = volumeCreditsFor(result);//volumeCreditsFor 함수를 statement의 중첩함수로 만들어서 중간 데이터 생성
        return result;
    }

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }

    function amountFor(aPerformance) {
        let result = 0;

        switch (aPerformance.play.type) {
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

    function volumeCreditsFor(aPerformance) {
        let volumeCredits = 0;

        volumeCredits += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" == aPerformance.play.type) // -> 변수 인라인 하기{
            volumeCredits += Math.floor(aPerformance.audience / 5);

        return volumeCredits;
    }

    function totalAmount(data) {
        return data.performances.reduce((total, p) => total + p.amount, 0);
    }

    function totalVolumeCredits(data) {
        return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
    }
}
```

이렇게 나누고 보니, 코드량이 처음보다 부쩍 는것을 볼 수 있습니다. 코드량이 단순히 늘어나기만 했다면 이것은 잘못된 것이지만, 코드량이 증가함으로써 얻은 이점이 많습니다. 코드의 논리적인 구조, 동작 단계를 명확히 구분할 수 있게 되었습니다. 이렇게 모듈화가 되면 각 부분이 하는 일과 그 부분들이 맞물려 돌아가는 과정을 파악하기 쉬워집니다.

이렇게 깔끔하게 코드를 유지하라고 저자는 당부합니다.

> 캠핑자들에게는 "도착했을 때보다 깔끔하게 정돈하고 떠난다" 는 규칙이 있다. 프로그래밍도 마찬가지다. 항시 코드베이스를 작업 시작 전보다 건강하게 만들어놓고 떠나야 한다.

어떻게보면 아주 당연한 말이지만, 저자가 이토록 강조하는 것은 당연한것을 지키지 않는 사람이 더 많기 때문이라고 생각합니다. 우선, 동작만 되면 다음에... 다음에... 하다가 결국 코드는 병들어 버리기 마련입니다. 물론 주어진 시간내에 프로젝트를 완료하기 위해서는 적당한 타협도 필요하겠지만, 기본은 코드를 깔끔하게 유지하기 가 되어야 한다고 생각합니다.





