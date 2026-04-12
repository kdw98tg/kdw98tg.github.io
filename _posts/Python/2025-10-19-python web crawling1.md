---
title: "[python] python 웹 크롤링 기초"

categories:
  -  Python
  
tags:
  - [python]

toc: true
toc_sticky: true

published: true

date: 2025-10-19
last_modified_at: 2024-10-19
---

## 웹 크롤링 이란?

웹 크롤링(Crawling)이란 말 그대로 “기어다니듯” 웹페이지를 돌아다니며 데이터를 모으는 작업을 말합니다. 이와 유사한 개념으로 웹 스크래핑(Scraping)이 있는데, 이는 “긁어모으다”라는 의미로 원하는 데이터를 추출하는 행위에 가깝습니다. 실무에서는 두 용어를 거의 같은 의미로 사용합니다.

웹 크롤링은 다양한 분야에서 사용됩니다. 사용되는 분야는 다음과 같습니다.
- **데이터 분석** : 대량의 웹 데이터를 수집하여 통계, 예측 등에 활용
- **AI 학습 데이터 구축** : 인공지능 모델에 필요한 텍스트, 이미지, 영상 데이터 수집
- **상품·콘텐츠 자동 업로드** : 쇼핑몰 상품 등록 자동화
- **투자 데이터 수집** : 부동산, 주식, 코인 등의 시세 데이터 수집
- **SNS 분석** : 인스타그램, 유튜브 트렌드 모니터링
- **뉴스·논문 크롤링** : 최신 기사나 학술자료 수집
- **구인광고 수집** : 채용 공고를 모아 분석

## 정적 웹 페이지, 동적 웹 페이지 

웹 페이지는 크게 두 가지로 나눌 수 있습니다.

정적 html 페이지는 서버에 완성된 html에서 내용들을 바로 가져올 수 있습니다. 그렇기 때문에, 단순히 html만 분석하면 됩니다.

반면 동적 html 페이지는 javascript가 실행된 후 데이터가 로딩됩니다. 이런 경우에는 Selenium, Playwright 같은 브라우저 자동화 도구가 필요하게 됩니다.

이번 포스팅에서는 정적 웹 html의 예제를 다룹니다.

## 웹 크롤링 기본 단계

웹 크롤링의 절차는 다음과 같습니다.

1. 데이터 받아오기 (request 라이브러리 사용하여 원하는 url의 html을 받아옴)
2. 데이터 가공하기 (가져온 html 문서를 BeatifulSoup 라이브러리를 활용해서 분석)

데이터를 받아오고, 데이터를 가공하는 형식으로 끝입니다.

## 페이징 처리하기

웹 페이지 (쇼핑몰 등)에서 쇼핑을 하다보면, 아이템이 워낙 많기 때문에 해당 아이템들을 하나의 페이지에서 표현하지 않습니다. 그때, 페이징 처리를 하여 아이템들을 보여주게 됩니다.

크롤링을 할 때, 페이징 처리된 모든 아이템을 가져오는것이 목적이므로, 모든 페이지를 돌며 html을 가져와야 합니다.

제가 이번에 강의를 보면서 강의자가 만들어 놓은 사이트에서는 `?page=1`과 같은 형태로 페이징 처리를 해놨었습니다.

따라서 반복문을 돌면서 각 페이지를 순서대로 요청하면 됩니다.


## 예시 코드

```python
import requests
from bs4 import BeautifulSoup

# 1~4 페이지 순회
for i in range(1, 5):
    response = requests.get(f"https://startcoding.pythonanywhere.com/basic?page={i}")
    html = response.text
    soup = BeautifulSoup(html, "html.parser")

    # 상품 정보 추출
    items = soup.select(".product")

    for item in items:
        category = item.select_one(".product-category").text
        name = item.select_one(".product-name").text
        link = item.select_one(".product-name > a").attrs['href']
        price = item.select_one(".product-price").text.split("원")[0]

        print(category, name, link, price)

```

코드의 시퀀스는 다음과 같습니다.

1. request 라이브러리를 통하여 http 통신을 합니다. 
2. 페이징 처리된 웹사이트의 모든 아이템을 가져오기 위하여 반복문을 돌면서 페이지를 모두 순회합니다.
3. 모든 페이지를 순회하며 html을 가져옵니다.
4. 가져온 페이지들 안에 있는 아이템들의 갯수 만큼 반복문을 돌리며 BeautifulSoup 라이브러리를 활용해 css 선택자를 기준으로 원하는 데이터를 가져옵니다.
5. 가져온 데이터를 출력합니다.

이렇게 하면 간단한 크롤링 예제가 만들어 집니다.

## 마무리

이번 포스팅에서는 단순히 데이터를 가져오는 방법에 대해서만 기술하였습니다. 또한, 상용 서비스 중인 정적 html은 html의 구조가 바뀌게 되면 코드도 바꿔야 하므로, 강의에서 사용한 예제 웹사이트를 사용하였습니다.

또한, 지금은 데이터를 저장하는 로직이 없는데, 다음 포스팅에서 mysql에 해당 데이터들을 저장하는 내용을 다룰 예정입니다.

마지막으로, 크롤링은 웹사이트 운영자에게 의존성이 있다는 생각이 들었습니다. 다음에 크롤링을 강의하게 되면, 제가 직접 실습용 웹사이트를 제작하여 수강생들이 크롤링을 연습할 수 있도록 해야겠다는 생각을 하였습니다.