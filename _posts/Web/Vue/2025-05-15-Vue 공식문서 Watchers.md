---
layout: post
title: "[vue/js] Vue 의 Watchers"

excerpt_image: /assets/images/posts/2026-05-13-Vue%20공식문서-bind/file-20260513223652607.png

categories:
  - Web
tags:
  - vue
  - javascript
toc: true
toc_sticky: true
published: true
date: 2026-05-15
last_modified_at: 2026-05-15
---

## Vue의 Watchers

해당 포스팅은 Vue 의 공식문서를 참조하여 작성하였습니다.

![](/assets/images/posts/2026-05-13-Vue%20공식문서-bind/file-20260513223652607.png)

반응형 웹에서는 때때로, 부수효과를 반응적으로 수행해야 할 필요가 있습니다. 예를 들어, `ref()`로 선언된 count 라는 변수의 숫자가 변경될때 마다 콘솔에 로그를 남기는 경우입니다.

좀더 추상적으로 이야기하자면, 반응형 변수가 변경될 때, 특정 js 로직이 돌아가야 하는 경우입니다. 실제로는 값이 변경될 때, 새로 get Method 를 실행하여 네트워크에서 데이터를 받아오는 경우가 될 수 있겠습니다.

react 에서는 해당 기능을 수행하는 hook 인 `useEffect()`를 사용합니다. vue 에서는 `watch()` 또는 `watcher()`를 사용하여 해당 기능을 구현합니다.

react에서의 `useEffect()`는 어떤 값이 바뀔 때, 다시 특정 동작을 수행하도록 `dependency array`를 넣도록 되어 있습니다. 또한, 컴포넌트가 마운트 될 때, 반드시 한번 `useEffect()`의 인자로 받은 함수가 한번 실행이 됩니다.

vue 에서 사용하는 `watch()`함수는 따로 `dependency array`를 넣을 필요 없이, 함수의 인자로 반응형 변수와 실행할 함수를 넣어주면 됩니다.

`watch()` 함수는 다음과 같이 사용합니다.

```vue
<script setup>
import { ref, watch } from 'vue';
const count = ref(0);

watch(count, (newCount) => {
    console.log(`new Count is ${newCount}`);
})

</script>
```

`watch()`함수의 특징은, 새로 변한 값을 함수에 인자로 던져주기 때문에, 바뀐 값을 가지고 작업을 할 수 있다는 점이 있습니다. 또한, react 의 `useEffect()`와는 다르게 컴포넌트 마운트시 실행이 안된다는 점이 있습니다. 따라서, 해당 로직을 실행하고 싶다면, js가 실행될 때, 명시적으로 선언하거나, onMounted 에서 실행해줘야 합니다. 또는, `immediate` 속성을 조작해서 컴포넌트 마운트시 바로 한번 실행하게 할 수 있습니다.

```vue
<script setup>
import { ref, watch } from 'vue';
const count = ref(0);

watch(count, (newCount) => {
    console.log(`new Count is ${newCount}`);
}, {immediate: true})

</script>
```

아래는 `watch()`함수를 사용하여 네트워크 통신을 하고, todo에 대한 새로운 정보를 가져오는 예제입니다.

```vue
<script setup>
//때로, 부수 효과를 반응적으로 수행해야 할 필요가 있음
//예를들어 숫자가 변경될 때마다 콘솔에 로그를 남기는 경우임
//이러한 작업은 watcher를 통해서 달성할 수 있음
//useEffect에 DependencyArray를 넣어서 사용하는것과 같은 맥락인듯

//인자로 넘겨준 count의 값이 바뀔 때 마다, 콜백이 실행됨.
import { ref, watch } from 'vue';
const count = ref(0);

const todoId = ref(1);
const todoData = ref(null);

watch(count, (newCount) => {
    console.log(`new Count is ${newCount}`);
})

watch(todoId, getTodo, { immediate: true })

async function getTodo() {
    todoData.value = null;
    const newTodoData = await fetch(`https://jsonplaceholder.typicode.com/todos/${todoId.value}`);
    todoData.value = await newTodoData.json();
}
function addTodoId() {
    ++todoId.value;
}

</script>

<template>
    <div>
        <button @click="count++">할일 Counter : {{ count }}</button>
    </div>

    <div>
        <button @click="addTodoId()">할일 가져오기 : {{ todoId }}</button>
        <p>{{ todoData }}</p>
    </div>
</template>

<style></style>
```

![](/assets/images/posts/2025-05-15-Vue%20공식문서%20Watchers/file-20260515214126806.png)

![](/assets/images/posts/2025-05-15-Vue%20공식문서%20Watchers/file-20260515214138160.png)