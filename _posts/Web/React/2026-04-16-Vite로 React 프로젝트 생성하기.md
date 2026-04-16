---
layout: post

excerpt_image: assets/images/posts/2026-04-16-Vite로%20React%20프로젝트%20생성하기/file-20260416124916878.png

title: "[React/Js] Vite로 React 앱 생성하기"

categories:
  -  Web
  
tags:
  - React
  - Vite

toc: true
toc_sticky: true

published: true

date: 2026-04-16
last_modified_at: 2026-04-16
---

## Vite

Vite는 한마디로 `엄청나게 빠른 차세대 프론트엔드 빌드 도구` 입니다. Vite가 빠른데에는 다음과 같은 이유가 있습니다.

- Native ESM 이용 : 브라우저가 직접 모듈을 요청하게 함. 서버는 전체를 빌드하지 않고, 브라우저가 요청하는 파일만 즉시 넘김
- Esbuild 활용 : 내부적으로 Go언어로 작성된 초고속 빌드 엔진인 Esbuild를 사용하여 의존성 라이브러리를 미리 빌드 함. 이게 Js 보다 수십 배 빠름

React 도 앞으로 CRA 에 대한 지원을 그만하고, Vite로 빌드하는것을 권장하고 있습니다.

## 프로젝트 생성하기

Vite 를 사용하려면 당연하게도 `Node Js` 설치가 필수입니다. (18+ 이상의 버전)

### npm 으로 빌드하기
```bash
# 일반적인 생성
npm create vite@latest

# 시작하면서 프로젝트 이름이나 템플릿을 지정할 때
npm create vite@latest my-react-app --template react
# 위 설정에서 ts 로 만들고 싶을 때
npm create vite@latest my-react-app --template react-ts
```

### yarn으로 빌드하기
```bash
yarn create vite
```

## 프로젝트 실행 및 빌드

프로젝트 생성이 완료 되었다면 다음과 같은 명령어로 프로젝트를 실행합니다.

### 개발모드 실행
``` bash
npm install
npm run dev
```

### 빌드 실행
```bash
npm run build
```


