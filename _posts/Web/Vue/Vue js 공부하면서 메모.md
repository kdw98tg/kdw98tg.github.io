---
publicshed: false
---

Vue는 프론트엔드 개발에 필요한 대부분의 일반적인 기능을 포괄하는 프레임워크이자 생태계입니다. 하지만 웹은 매우 다양하며, 우리가 웹에서 만드는 것들은 형태와 규모가 크게 다를 수 있습니다. 이를 염두에 두고, Vue는 유연하고 점진적으로 도입할 수 있도록 설계되었습니다. 사용 사례에 따라 Vue는 다양한 방식으로 사용할 수 있습니다:

- 빌드 단계 없이 정적 HTML 강화
- 어떤 페이지에도 웹 컴포넌트로 임베딩
- 싱글 페이지 애플리케이션(SPA)
- 풀스택 / 서버 사이드 렌더링(SSR)
- Jamstack / 정적 사이트 생성(SSG)
- 데스크탑, 모바일, WebGL, 심지어 터미널까지 타겟팅


## 싱글 파일 컴포넌트[​](https://ko.vuejs.org/guide/introduction#single-file-components)

대부분의 빌드 도구 기반 Vue 프로젝트에서는 **싱글 파일 컴포넌트**(일명 `*.vue` 파일, **SFC**로 약칭)라는 HTML 유사 파일 형식을 사용하여 Vue 컴포넌트를 작성합니다. Vue SFC는 이름 그대로 컴포넌트의 로직(JavaScript), 템플릿(HTML), 스타일(CSS)을 하나의 파일에 캡슐화합니다. 아래는 앞서 본 예시를 SFC 형식으로 작성한 것입니다:

vue

```
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

SFC는 Vue의 대표적인 기능이며, **빌드 설정이 필요한 경우** Vue 컴포넌트를 작성하는 권장 방식입니다.

-> 리엑트는 jsx 인데 vue 는 vue 라는 확장자를 씀


## 옵션 api, 컴포지션 api
옵션 api 는 영역나눠서 this로 참조
컴포지션 api 는 리액트처럼 ref 나 이런 훅을 사용해서 참조


### DOM 내 루트 컴포넌트 템플릿[​](https://ko.vuejs.org/guide/essentials/application.html#in-dom-root-component-template)

루트 컴포넌트의 템플릿은 보통 컴포넌트 자체의 일부이지만, 마운트 컨테이너 내부에 직접 작성하여 별도로 템플릿을 제공할 수도 있습니다:

html

```
<div id="app">
  <button @click="count++">{{ count }}</button>
</div>
```

js

```
import { createApp } from 'vue'

const app = createApp({
  data() {
    return {
      count: 0
    }
  }
})

app.mount('#app')
```

루트 컴포넌트에 이미 `template` 옵션이 없다면, Vue는 자동으로 컨테이너의 `innerHTML`을 템플릿으로 사용합니다.

DOM 내 템플릿은 [빌드 단계 없이 Vue를 사용하는](https://ko.vuejs.org/guide/quick-start.html#using-vue-from-cdn) 애플리케이션에서 자주 사용됩니다. 또한 서버 사이드 프레임워크와 함께 사용할 수도 있으며, 이 경우 루트 템플릿이 서버에서 동적으로 생성될 수 있습니다.

-> SSR 기반으로 만들때 이렇게 한번에 빌드없이 사용 가능 (App.vue 대신에 js 에 한번에 다 만드는 과정임)

```vue
<script setup>
import { ref, reactive } from 'vue'

// 컴포넌트 로직
// 여기에 반응형 상태를 선언합니다.
  //reactive 는 객체 및 배열 (Map, Set)과 같은 타입에만 동작함
  //반면, ref() 는 어떤 값이든 받아 내부 값을 .value 속성으로 노출하는 객체 생성 가능
  const counter = reactive({
    count: 0
  });
  
  console.log(counter);
  counter.count++;
  
  const message = ref("Hello, World!");
  console.log(message.value)// "Hello, World!"
  message.value = "Changed"
</script>

<template>
  <!-- 템플릿에서 ref에 접근할때는 .value 없어도 됨 -->
<h1>{{ message }}</h1>
<p>Count is: {{ counter.count }}</p>
  <h1>{{ message.split('').reverse().join('') }}</h1>

</template>
```

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment(){
  ++count.value;
}
</script>

<template>
  <!-- 이 버튼이 작동하도록 만들어 봅시다 -->
<!-- 여기서 increment는 setup 에서 선언된 함수 이름을 따라감    -->
  <button v-on:click="increment">숫자 세기: {{ count }}</button>
  <!-- v-on 은 자주사용되기 때문에, 축약 문법도 존재함 -->
  <button @click="increment">{{count}}</button>
</template>
```

```vue
<script setup>
import { ref } from 'vue'

const awesome = ref(true)

function toggle() {
  // ...
  awesome.value = !awesome.value;
}
</script>

<template>
  <button @click="toggle">토글 버튼</button>
  <h1 v-if="awesome">Vue는 굉장해! 엄청나!</h1>
  <h1 v-else>오 안돼 😢</h1>
</template>
```

```js
<script setup>
import { ref } from 'vue'

// 각 할 일에 고유한 ID 부여
let id = 0

const newTodo = ref('')
const todos = ref([
  { id: id++, text: 'HTML 배우기' },
  { id: id++, text: 'JavaScript 배우기' },
  { id: id++, text: 'Vue 배우기' }
])

function addTodo() {
  // ...
  if(newTodo.value ==='') return;
  todos.value.push({id: id++, text: newTodo.value});
  newTodo.value = '';
}

function removeTodo(todo) {
  // ...
  todos.value = todos.value.filter((t) => t !== todo)
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo" required placeholder="new todo">
    <button @click="addTodo">할 일 추가</button>
  </form>
  <ul>
    <li v-for="todo in todos" :key="todo.id">
      {{ todo.text }}
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
</template>
```