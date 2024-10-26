---
title: "[Refactoring] 리팩터링 2판 Chapter.01 (다형성을 활용해 계산 코드 재구성하기)"

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

이번 포스팅에서는 연극 장르를 추가하고, 장르마다 공연료와 적립 포인트 계산법을 다르게 지정하도록 기능을 수정 합니다. 

가장 쉬운 방법으로는 함수 안에 조건을 추가하여 구현하는 것입니다. 하지만, 이렇게 하면 조건이 들어가야하는 함수를 전부 찾아 바꿔주야 합니다. 관리포인트가 엄청 늘어나서 언젠가는 버그가 나서 골머리를 앓게 될것입니다.

그렇기 때문에 객체지향의 `다형성`을 사용하여 리팩터링 합니다. (js 는 원래 객체지향 기능이 없었지만, ES6 버전부터 객체지향을 사용할 수 있게 되었습니다.)

저자는 해당 기법을 `조건부 로직을 다형성으로 바꾸기`라고 이름 지었습니다.

우선, 객체지향의 다형성인 만큼 클래스를 생성 합니다. 계산을 해주는 계산기 클래스 이름은 `PerformanceCalculator`라고 하겠습니다. 

```js
class PerformanceCalculator {
    constructor(aPerformance, aPlay) {
        this.performances = aPerformance;
        this.play = aPlay;
    }
}

export default function createStatementData(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);

    function enrichPerformance(aPerformance) {
        const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance));//계산기 생성
        const result = Object.assign({}, aPerformance);
        result.play = playFor(result);
        result.amount = amountFor(result);
        result.volumeCredits = volumeCreditsFor(result);
        return result;
    }

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }

    function amountFor(aPerformance) {
        let result = 0;

        switch (aPerformance.play.type) {
            case "tragedy":
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

        return result;
    }

    function volumeCreditsFor(aPerformance) {
        let volumeCredits = 0;

        volumeCredits += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" == aPerformance.play.type)
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

이제, 함수들을 `PerformanceCalculator` 클래스로 옮기는 작업을 합니다.

```js
class PerformanceCalculator {
    constructor(aPerformance, aPlay) {
        this.performance = aPerformance;
        this.play = aPlay;
    }

    get amount() {
        let result = 0;

        switch (this.play.type) {
            case "tragedy":
                thisAmount = 4000;
                if (aPerformance.audience > 30) {
                    thisAmount += 1000 * (aPerformance.audience - 30);
                }
                break;
            case "comedy":
                thisAmount = 3000;
                if (this.performance.audience > 20) {
                    thisAmount += 10000 + 500 * (this.performance.audiece - 20);
                }
                thisAmount += 300 * this.performance.audience;
                break;

            default:
                throw new Error('알 수 없는 장르: ${this.play.type}');
        }

        return result;
    }
}

export default function createStatementData(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);

    function enrichPerformance(aPerformance) {
        const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance));//계산기 생성
        const result = Object.assign({}, aPerformance);
        result.play = playFor(result);
        result.amount = amountFor(result);
        result.volumeCredits = volumeCreditsFor(result);
        return result;
    }

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }

    function amountFor(aPerformance) {
       return new PerformanceCalculator(aperformance, playFor(aPerformance)).amount;//클래스의 함수 이용하기
    }

    function volumeCreditsFor(aPerformance) {
        let volumeCredits = 0;

        volumeCredits += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" == aPerformance.play.type)
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

우선, `amountFor()` 함수를 `PerformanceCalculator` 클래스에 옮기는 작업을 하였습니다. 처음에는 `amountFor()` 함수 안에 클래스를 생성하여 돌리도록 하고, 저자가 강조하는 `컴파일 - 테스트 - 커밋` 절차를 통과 하였다면, 해당 `함수를 인라인` 하도록 합니다.  (volumeCredit() 함수도 인라인 한 코드를 아래에 작성 해놨습니다.)

```js
class PerformanceCalculator {
    constructor(aPerformance, aPlay) {
        this.performance = aPerformance;
        this.play = aPlay;
    }

    get amount() {
        let result = 0;

        switch (this.play.type) {
            case "tragedy":
                thisAmount = 4000;
                if (aPerformance.audience > 30) {
                    thisAmount += 1000 * (aPerformance.audience - 30);
                }
                break;
            case "comedy":
                thisAmount = 3000;
                if (this.performance.audience > 20) {
                    thisAmount += 10000 + 500 * (this.performance.audiece - 20);
                }
                thisAmount += 300 * this.performance.audience;
                break;

            default:
                throw new Error('알 수 없는 장르: ${this.play.type}');
        }

        return result;
    }

    get volumeCredits() {
        let volumeCredits = 0;

        volumeCredits += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" == aPerformance.play.type)
            volumeCredits += Math.floor(aPerformance.audience / 5);

        return volumeCredits;
    }
}

export default function createStatementData(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);

    function enrichPerformance(aPerformance) {
        const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance));//계산기 생성
        const result = Object.assign({}, aPerformance);
        result.play = playFor(result);
        result.amount = calculator.amount;//함수를 클래스로 이동시킨 후, 인라인
        result.volumeCredits = calculator.volumeCredits;//함수를 클래스로 이동시킨 후, 인라인
        return result;
    }

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }

    function totalAmount(data) {
        return data.performances.reduce((total, p) => total + p.amount, 0);
    }

    function totalVolumeCredits(data) {
        return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
    }
}
```

`PerformanceCalculator`클래스가 완성되었습니다. 이제, 해당 클래스를 부모 클래스로 하는 서브 클래스들을 만들어서 `다형성`을 지원하도록 수정해 봅시다.자바스크립트는 부모 클래스에서 자식의 인스턴스를 반환할 수 없다고 합니다. 그래서 팩터리 함수를 만들어서 사용해야 합니다. `생성자를 팩터리 함수로 바꾸기` 기법을 적용 합니다.

클래스는 `TragedyCalculator` 와 `ComedyCalculator`로 구성합니다.

```js
class PerformanceCalculator {
    constructor(aPerformance, aPlay) {
        this.performance = aPerformance;
        this.play = aPlay;
    }

    get amount() {
        let result = 0;

        switch (this.play.type) {
            case "tragedy":
                thisAmount = 4000;
                if (aPerformance.audience > 30) {
                    thisAmount += 1000 * (aPerformance.audience - 30);
                }
                break;
            case "comedy":
                thisAmount = 3000;
                if (this.performance.audience > 20) {
                    thisAmount += 10000 + 500 * (this.performance.audiece - 20);
                }
                thisAmount += 300 * this.performance.audience;
                break;

            default:
                throw new Error('알 수 없는 장르: ${this.play.type}');
        }

        return result;
    }

    get volumeCredits() {
        let volumeCredits = 0;

        volumeCredits += Math.max(aPerformance.audience - 30, 0);
        if ("comedy" == aPerformance.play.type)
            volumeCredits += Math.floor(aPerformance.audience / 5);

        return volumeCredits;
    }
}

class TragedyCalculator extends PerformanceCalculator {

}

class ComedyCalculator extends PerformanceCalculator {

}

function createPerformanceCalculator(aPerformance, aPlay) {
    switch (aPlay.type) {
        case "tragedy": return new TragedyCalculator(aPerformance, aPlay);
        case "comedy" : return new ComedyCalculator(aPerformance, aPlay);
        default : throw new Error("알 수 없는 장르 : ${aPlay.type}");
    }
}

export default function createStatementData(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);

    function enrichPerformance(aPerformance) {
        const calculator = createPerformanceCalculator(aPerformance, playFor(aPerformance));//계산기 생성
        const result = Object.assign({}, aPerformance);
        result.play = playFor(result);
        result.amount = calculator.amount;//함수를 클래스로 이동시킨 후, 인라인
        result.volumeCredits = calculator.volumeCredits;//함수를 클래스로 이동시킨 후, 인라인
        return result;
    }

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }

    function totalAmount(data) {
        return data.performances.reduce((total, p) => total + p.amount, 0);
    }

    function totalVolumeCredits(data) {
        return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
    }
}
```

이제, 새로 만든 클래스 안에 `조건부 로직을 다형성으로 바꾸기`를 적용하기 위한 함수를 정의 합니다. `amount()`의 복잡한 switch 문도 이제 필요없게 되었습니다. 또한 `volumeCredits` 도 각 클래스에서 정의하면 되기 때문에 부모클래스에서는 필요없게 되었습니다.

```js
class PerformanceCalculator {
    constructor(aPerformance, aPlay) {
        this.performance = aPerformance;
        this.play = aPlay;
    }

    get amount() {
        throw new Error('서브클래스에서 이용하세요.');
    }
}

class TragedyCalculator extends PerformanceCalculator {
    get amount() {
        let result = 40000;
        if (this.performance.audience > 30) {
            result += 1000 * (this.performance.audience - 30);
        }
        return result;
    }

    get volumeCredits() {
        return Math.max(this.performance.audience - 30, 0);
    }
}

class ComedyCalculator extends PerformanceCalculator {
    get amount() {
        let result = 30000;
        if (this.performance.audience > 20) {
            result += 10000 + 500 * (this.performance.audience - 20);
        }
        result += 300 * this.performance.audience;
        return result;
    }

    get volumeCredits() {
        let volumeCredits = Math.max(this.performance.audience - 30, 0);
        volumeCredits += Math.floor(this.performance.audience / 5);
        return volumeCredits
    }
}

function createPerformanceCalculator(aPerformance, aPlay) {
    switch (aPlay.type) {
        case "tragedy": return new TragedyCalculator(aPerformance, aPlay);
        case "comedy": return new ComedyCalculator(aPerformance, aPlay);
        default: throw new Error("알 수 없는 장르 : ${aPlay.type}");
    }
}

export default function createStatementData(invoice, plays) {
    const statementData = {};
    statementData.customer = invoice.customer;
    statementData.performances = invoice.performances.map(enrichPerformance);
    statementData.totalAmount = totalAmount(statementData);
    statementData.totalVolumeCredits = totalVolumeCredits(statementData);

    function enrichPerformance(aPerformance) {
        const calculator = createPerformanceCalculator(aPerformance, playFor(aPerformance));//계산기 생성
        const result = Object.assign({}, aPerformance);
        result.play = playFor(result);
        result.amount = calculator.amount;//함수를 클래스로 이동시킨 후, 인라인
        result.volumeCredits = calculator.volumeCredits;//함수를 클래스로 이동시킨 후, 인라인
        return result;
    }

    function playFor(aPerformance) {
        return plays[aPerformance.playID];
    }

    function totalAmount(data) {
        return data.performances.reduce((total, p) => total + p.amount, 0);
    }

    function totalVolumeCredits(data) {
        return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
    }
}

```

여기까지 chap.1 의 예제가 끝이 났습니다. 처음코드와 비교하면 코드 라인수는 확실히 늘었지만, 기능별, 단계별로 정리가 잘 되어있어서 유지보수하기가 훨씬 간결하고, 그냥 코드를 읽을때도 더 가독성이 좋아졌습니다.

지금까지의 리팩터링을 정리하면, 크게 3단계로 구성되어 있습니다.
`원본 중첩함수를 여러개로 나누기` -> `단계 쪼개기` -> `계산코드와 출력코드 분리` 의 순서 입니다.

리팩터링은 대부분 코드가 하는 일을 파악하는 데서 시작합니다. 그래서 코드를 읽고, 개선점을 찾고, 반영하는 식으로 진행 합니다. 

저자는 다음과 같은 말을 남깁니다.

> 좋은 코드를 가늠하는 확실한 방법은 '얼마나 수정하기 쉬운가?' 다.

개발자마다 좋은 코드가 무엇인지에 대해 말하는 기준은 다릅니다. 각 개발자의 취향과 경험에 따라 달라질 수 있는거죠. 하지만 `좋은 코드`의 본질은 명확하다고 생각합니다. `얼마나 수정하기 쉬운가?` 입니다. 소프트웨어를 공부하는 사람이든, 아니든 누구든 소프트웨어를 한번이라도 사용해 보았다면 `ver.1`로 배포가 완료된 완벽한 소프트웨어란 존재하지 않습니다. 그렇다면, 코드를 작성할 때 개선될 부분을 생각하면서 작성해야 합니다. 미래를 예측하는것은 불가능하지만, 미래를 대비하는것은 할 수 있습니다. 결국 `리팩터링이란 불확실한 미래를 대비하는 작업` 이라고 할 수 있겠습니다.

마지막으로, 저자는 효과적으로 리팩터링 하는 방법을 말합니다.

> 핵심은, 단계를 잘게 나눠야 더 빠르게 처리할 수 있고, 코드는 절대 깨지지 않으며, 이러한 작은 단계들이 모여서 상항히 큰 변화를 이룰 수 있다는 사실을 깨닫는 것이다. 이 점을 명심하고 그대로 따라주기 바란다.

잠시 다른이야기를 하자면, 저는 성공에 굉장히 관심이 많습니다. 제 유튭에도 보면 성공과 관련된 영상들이 많이 올라옵니다. 책도 많이 보구요. 책이든 유튜브 영상이든 다들 똑같이 하는 이야기들이 있습니다.` 급하지 않게, 한단계식 쌓아가라` 라는 이야기 입니다. 하루에 1% 씩 성장할 수 있으면, 1년에 37배 성장할 수 있다고 합니다. 하지만, 단계를 밟지 않고 뛰어넘으려고 하면 결국 무너저 다시 처음부터 하게되는 불상사를 겪게 될것입니다. (맨날 속으로 되내이지만 성격이 급해서 정말 지키기 어렵습니다..) 

이야기가 샜지만, 리팩터링도 같은 원리인 것 같습니다. 모듈단위로 보면 모듈을 깨끗하게 정리하는것, 프로젝트로 보면 모듈들을 깔끔하게 정리하는 것이 불확실한 미래에 대비하여 프로젝트를 지속가능하게 만들 수 있는 것 같습니다. 다시한번 되세깁니다. `작은 단계들이 모여 큰 변화를 이룬다!`



