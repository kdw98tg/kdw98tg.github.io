---
layout: post
title: "[vue/js] Vite의 Mustahce를 사용하여 html에 값 표현하기"
excerpt_image: assets/images/posts/2026-05-13-%20Vue%20공식문서%20Mustache/file-20260513222057351.png
categories:
  - Web
tags:
  - vue
  - javascript
  - vite
toc: true
toc_sticky: true
published: true
date: 2026-05-13
last_modified_at: 2026-05-13
---

## Vue의 Mustache 문법

해당 포스팅은 Vue 의 공식문서를 참조하여 작성하였습니다.

![](/assets/images/posts/2026-05-13-%20Vue%20공식문서%20Mustache/file-20260513222057351.png)

React나 Vue 처럼 front-end 프레임워크는 필히 js의 변수의 값을 html로 렌더링 하는 경우가 많습니다.

이때, Vue 에서는 `{{}}` 와 같은 기호를 씁니다.

이 기호는 사람의 콧수염을 닮았다고 하여 `Mustache`라고 부릅니다.

Vue의 Mustache는 텍스트를 html로 나타내기 위하여 사용하는 문법입니다. 

```vue
<script setup>
  import {reactive, ref} from 'vue';
  const counter = reactive({count: 1});
  const message = ref("안녕하세요")
</script>

<template>
  <h1>{{ message }}</h1>
  <p>숫자 세기 : {{ counter.count }}</p>
</template>

<style scoped></style>
```

해당 코드처럼, js로 선언된 counter와 message를 html로 나타내기 위해 사용합니다.

![](/assets/images/posts/2026-05-13-%20Vue%20공식문서%20Mustache/file-20260513222356878.png)


