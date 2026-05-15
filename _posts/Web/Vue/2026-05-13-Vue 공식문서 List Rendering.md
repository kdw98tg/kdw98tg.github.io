---
layout: post
title: "[vue/js] Vue 의 html에서 List를 Rendering 하기 v-for"

excerpt_image: assets/images/posts/2026-05-13-Vue%20공식문서-bind/file-20260513223652607.png

categories:
  - Web
tags:
  - vue
  - javascript
toc: true
toc_sticky: true
published: true
date: 2026-05-13
last_modified_at: 2026-05-13
---

## Vue에서 List 를 렌더링 하기

해당 포스팅은 Vue 의 공식문서를 참조하여 작성하였습니다.

![](/assets/images/posts/2026-05-13-Vue%20공식문서-bind/file-20260513223652607.png)

Vue 에서는 `v-if`와 같은 편리한 기능을 제공하는 디렉티브가 있습니다. 리스트를 렌더링할때 또한, `v-for`라는 디렉티브가 있습니다.

해당 디렉티브는 forEach 와 같은 문법으로 작성됩니다. 사용방법은 다음과 같습니다.

```vue
<ul>
  <li v-for="todo in todos" :key="todo.id">
    {{ todo.text }}
  </li>
</ul>
```

해당 구문을 사용하여, 할일 목록 리스트를 만드는 예제를 구현합니다.

```vue
<script setup>
import { ref } from 'vue'

let id = 0;

const newTodo = ref('');
const todos = ref([
    { id: id++, text: "HTML 배우기" },
    { id: id++, text: "js 배우기" },
    { id: id++, text: "ts 배우기" },
]);

function addTodo() {
    todos.value.push({ id: id++, text: newTodo.value })
    newTodo.value = '';
}

function removeTodo(todo) {
    const newTodoArray = todos.value.filter(t => t.id !== todo.id);
    todos.value = newTodoArray;
}
</script>

<template>
    <div>
        <input type="text" v-model="newTodo">
        <button @click="addTodo">할 일 추가</button>
    </div>

    <ul>
        <li v-for="todo in todos" :key="todo.id">
            <span>{{ todo.text }}</span>

            <button @click="removeTodo(todo)">x</button>
        </li>
    </ul>
</template>

<style></style>

```

![](/assets/images/posts/2026-05-13-Vue%20공식문서%20List%20Rendering/file-20260513233649790.png)

![](/assets/images/posts/2026-05-13-Vue%20공식문서%20List%20Rendering/file-20260513233706798.png)
