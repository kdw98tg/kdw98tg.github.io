---
title: "[Refactoring] 리팩터링 2판 Chapter.06 part 4 (기본적인 리팩터링)"

categories:
  -  Refactoring
  
tags:
  - [Refactoring, javascript]

toc: true
toc_sticky: true

published: true

date: 2024-11-24
last_modified_at: 2024-11-24
---

## 단계 쪼개기

저자는 코드를 봤을 때, 두가지 일을 하는 함수를 보면 각각을 별개 모듈로 나누는 방법을 모색한다고 합니다. 나중에 코드를 수정해야할 일이 생겼을 때 두 대상을 동시에 생각할 필요 없이, 하나에만 집중하기 위해서라고 합니다.

이 말에 공감하는것이, 하나의 함수에 너무 많은 일을 집어 넣으면, 함수 이름을 짜기도 힘들 뿐더러 몇일만 지나도 그 함수를 다시 해석하여 코드를 수정해야 하기 때문입니다. 기능별로  분리하여 작성하면, 문제가 있는 부분만 살펴보면 됩니다.

이것이 잘 적용되어 있는곳이 컴파일러라고 합니다. 초기의 컴파일러는 사람이 친 텍스트를 기계어로 한번에 변환했다고 합니다. 컴파일러가 발전해 오면서 사람들은 컴파일 작업을 여러 단계가 순차적으로 연결된 형태로 분리하는것이 좋다는것을 깨달았습니다 (텍스트 토큰화 -> 토큰을 파싱하여 구문트리 만들기 -> 구문트리 최적화 -> 목적코드 생성의 순서). **복잡한 과정이지만, 각 단계는 자신만의 문제에 집중하기 떄문에 나머지 단계에 관해서는 자세히 몰라도 수정을 할 수 있습니다.**

### 절차
1. 두 번쨰 단계에 해당하는 코드를 독립 함수로 추출한다.
2. 테스트한다.
3. 중간 데이터 구조를 만들어서 앞에서 추출한 함수의 인수로 추가한다.
4. 테스트한다.
5. 추출한 두 번째 단계 함수의 매개변수를 하나씩 검토한다. 그중 첫 번째 단계에서 사용되는 것은 중간데이터 구조로 옮긴다. 하나씩 옮길 떄마다 테스트한다.
6. 첫 번째 단계 코드를 함수로 추출하면서 중간 데이터 구조를 반환하도록 만든다.

### 예제

다음은 상품의 금액을 계산하는 코드입니다.

```javascript
function priceOrder(product, quantity, shippingMethod) {

	//상품의 가격을 계산
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0)
        * product.basePrice * product.discountRate;
    
    
    //배송비를 계산
    const shippingPerCase = (basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountedFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost;
    return price;
}
```

이 코드를 보면, 2단계가 하나의 함수에 2가지의 기능이 들어가있는것을 볼 수 있습니다. 위쪽의 코드는 상품의 가격을 계산하는 코드, 아래쪽의 코드는 배송비를 계산하고 있습니다. 이 2단계를 분리하도록 리팩터링을 합니다.

우선, 배송비 관련 코드를 함수로 추출합니다.

```javascript
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0)
        * product.basePrice * product.discountRate;
    const price = applyShipping(price, shippingMethod, quantity, discount);
    return price;
}

function applyShipping(price, shippingMethod, quantity, discount) {
    const shippingPerCase = (price > shippingMethod.discountThreshold) ? shippingMethod.discountedFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost;
    return price;
}
```

지금 보면, applyShipping 함수의 인자가 벌써 4개가 된 것을 알 수 있습니다. 실제로 코드를 짜면 더 많아질 수 있습니다. 해당 악취는 `변수 캡슐화하기`로 처리할 수 있습니다. 따라서 중간 데이터를 만듭니다.

```javascript
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0)
        * product.basePrice * product.discountRate;
    const priceData = { basePrice: basePrice, quantity: quantity, discount: discount };
    const price = applyShipping(priceData, shippingMethod);
    return price;
}

function applyShipping(priceData, shippingMethod) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountedFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    const price = priceData.basePrice - priceData.discount + shippingCost;
    return price;
}
```

위와같이 만들 수 있습니다. 여기서, 앞선 상품의 가격을 계산하는 코드도 함수로 추출하면 다음과 같이 만들 수 있습니다.

```javascript
function priceOrder(product, quantity, shippingMethod) {
    const priceData = calculatePricingData(product, quantity);
    return applyShipping(priceData, shippingMethod);
}

function calculatePricingData(product, quantity) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0)
        * product.basePrice * product.discountRate;
    return { basePrice: basePrice, quantity: quantity, discount: discount };
}

function applyShipping(priceData, shippingMethod) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountedFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    const price = priceData.basePrice - priceData.discount + shippingCost;
    return price;
}
```

그리고 함수 이름을 통해 해당 함수가 반환해야 할 데이터가 정해졌으므로, `price`라는 지역변수를 인라인 해서 최종 마무리를 합니다.

```javascript
function priceOrder(product, quantity, shippingMethod) {
    const priceData = calculatePricingData(product, quantity);
    return applyShipping(priceData, shippingMethod);
}

function calculatePricingData(product, quantity) {
    const basePrice = product.basePrice * quantity;
    const discount = Math.max(quantity - product.discountThreshold, 0)
        * product.basePrice * product.discountRate;
    return { basePrice: basePrice, quantity: quantity, discount: discount };
}

function applyShipping(priceData, shippingMethod) {
    const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountedFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    return priceData.basePrice - priceData.discount + shippingCost;
}
```

이렇게하면 단계나누기 리팩터링이 완료되었습니다.