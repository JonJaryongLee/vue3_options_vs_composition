# Vue 3 - Options API  vs  Composition API

이 문서는 Vue 2 방식인 Options API 와, Vue 3 권장방식인 Composition API 의 비교를 위해 제작됨.

Vue 2 의 지원 종료일인 2023년 12월 31일 이후로도 Options API 는 사용 가능하다. 따라서, 기존의 Vue.js 코드를 전체 변경할 필요는 없다.

그러나, 신규 프로젝트는 Composition API 로 진행하길 강하게 권장한다.

Options API 를 계속 사용하길 결정했다 하더라도, Vue 3 에서의 변경점을 알아보기 위해 본 문서를 읽어보길 권장한다.

Vue 3 공식문서는 `<script>` 가 `<template>` 보다 먼저 작성되기에 이 서술에 따르겠다.

# 1. Basic Code

Options API

```html
<script>
export default {
  data() {
    return {
      message: "Hello World!",
      count: 0
    };
  },
  created() {
    this.message = "환영합니다";
    this.count = 1;
  }
};
</script>

<template>
  <h1>{{ message }}</h1>
  <p>count is: {{ count }}</p>
</template>
```

상태는 `data() { return { } }` 안에 선언했다.

컴포넌트 생성 이후에 메서드 없이 즉시 상태를 변경하고 싶을 경우, 반드시 `created` 를 사용했어야만 했다.

Composition API

```html
<script setup>
import { ref } from 'vue';
const message = ref("Hello World!");
const count = ref(0);

message.value = "환영합니다";
count.value = 1;
</script>

<template>
  <h1>{{ message }}</h1>
  <p>count is: {{ count }}</p>
</template>
```

`<script>` 태그는 `setup` 을 반드시 붙여준다.

`export default` 구문이 사라졌으며, `.js` 코드 작성할때처럼 편하게 작성해나가면 된다.

상태는 `vue` 에서 제공하는 `ref` 을 가져와서 선언하며, 매개변수로 초기값을 담는다.

- Vue 3 에 익숙해지기전까지 매우 혼란스러운 개념이지만, 반드시 알고 있어야 한다.

    `<script setup>` 안에서 상태에 접근할 땐 반드시 `.value` 를 붙여야 한다.

    `<template>` 안에서 상태에 접근할 땐 `.value` 를 붙이지 않는다.

더이상 `created` 는 사용하지 않는다. 데이터 패치 역시 마찬가지다. 해당 항목에서 다시 설명하겠다.

- Options API 와 Composition API 공통사항으로, 더이상 `<div>` 등의 루트 엘리먼트를 사용하지 않아도 된다.

# 2. Child Component

Options API

```html
<script>
import MyComponent from './MyComponent.vue'
export default {
  components: {
    MyComponent
  }
};
</script>

<template>
  <MyComponent />
</template>
```

Composition API

```html
<script setup>
import MyComponent from './MyComponent.vue'
</script>

<template>
  <MyComponent />
</template>
```

자식 컴포넌트 등록 시, `components` 옵션을 사용하는 과정이 생략되었다.

공통사항으로, 컴포넌트 사용 시 kebab-case 는 권장되지 않는다. 반드시 PascalCase 를 사용해 컴포넌트임을 명확히 표현해야 한다.

또한, 컴포넌트는 slot 을 사용하지 않는 이상 즉시 닫아준다.

나쁜 예시

```html
<sample-comp></sample-comp>
```

좋은 예시

```html
<SampleComp />
```

# 3. methods ⇒ function

Options API

```html
<script>
export default {
  data() {
    return {
      count: 0
    };
  },
  methods: {
    increment() {
      this.count++;
    }
  }
};
</script>

<template>
  <button @click="increment">count is: {{ count }}</button>
</template>
```

메서드는 `methods` 안에 정의한다. 화살표 함수는 `this` 의 scope 때문에 사용할 수 없었다.

Composition API

```html
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++;
}
</script>

<template>
  <button @click="increment">count is: {{ count }}</button>
</template>
```

`methods` 가 사라지고, `function` 키워드로 함수 선언한다.

화살표 함수도 사용 가능하지만, 함수임을 확실하게 하기 위해 `function` 키워드를 사용하는 선언이 권장된다.

- Composition API 의 가장 큰 장점: 객체가 아니기 때문에 더이상 `this` 를 사용할 필요 없음

# 4. computed

Todo List 에서, `hide completed` 버튼을 누르면 숨겨지고, `show all` 을 누르면 전체 리스트 출력하는 예제이다.

Options API

```html
<script>
export default {
  data() {
    return {
      id: 0,
      newTodo: '',
      hideCompleted: false,
      todos: [
        { id: id++, text: 'Learn HTML', done: true },
        { id: id++, text: 'Learn JavaScript', done: true },
        { id: id++, text: 'Learn Vue', done: false }
      ]
    };
  },
  computed: {
    filteredTodos() {
      if (this.hideCompleted) {
        return this.todos.filter((todo) => !todo.done);
      }
      return this.todos;
    }
  },
  methods: {
    addTodo() {
      this.todos.push({ id: this.todos.length, text: this.newTodo, done: false });
      this.newTodo = '';
    },
    removeTodo(todo) {
      this.todos = this.todos.filter((t) => t !== todo);
    }
  }
};
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo">
    <button>Add Todo</button>
  </form>
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      <input type="checkbox" v-model="todo.done">
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
  <button @click="hideCompleted = !hideCompleted">
    {{ hideCompleted ? 'Show all' : 'Hide completed' }}
  </button>
</template>

<style scoped>
.done {
  text-decoration: line-through;
}
</style>
```

`computed` 옵션을 사용해 구현한다.

Composition API

```html
<script setup>
import { ref, computed } from 'vue'

let id = 0

const newTodo = ref('')
const hideCompleted = ref(false)
const todos = ref([
  { id: id++, text: 'Learn HTML', done: true },
  { id: id++, text: 'Learn JavaScript', done: true },
  { id: id++, text: 'Learn Vue', done: false }
])

const filteredTodos = computed(() => {
  if (hideCompleted.value) {
    return todos.value.filter((todo) => !todo.done);
  }
  return todos.value;
})

function addTodo() {
  todos.value.push({ id: id++, text: newTodo.value, done: false })
  newTodo.value = ''
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t !== todo)
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo">
    <button>Add Todo</button>
  </form>
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      <input type="checkbox" v-model="todo.done">
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
  <button @click="hideCompleted = !hideCompleted">
    {{ hideCompleted ? 'Show all' : 'Hide completed' }}
  </button>
</template>

<style scoped>
.done {
  text-decoration: line-through;
}
</style>
```

`computed` 옵션이 사라지고, `vue` 에서 제공하는 `computed` 를 가져와 사용한다. 변수를 `computed` 로 정의하되, 매개변수로 콜백함수를 넣어준다.

- `id` 를 주목하자. 기존 Options API 는 화면의 변경사항과 관련 없는 항목들까지 모두 상태로 선언했다. 그러나 Composition API 에선 상황에 따라, 상태로 선언하지 않아도 되는 것은 일반변수로 선언하는것이 가능하다. 대표적으로 데이터 패치 시 `URL` 은 변하지 않는 경우가 대부분이기에 일반변수로 선언하기 적합하다. 말할것도 없이, 상태와 비교했을 때 일반 변수는 메모리 소모가 훨씬 적다.

# 5. Data Fetch

Options API

```html
<script>
import axios from 'axios';

export default {
  data() {
    return {
      todos: []
    };
  },
  created() {
    this.getTodos();
  },
  methods: {
    async getTodos() {
      const { data } = await axios.get(
        'https://jsonplaceholder.typicode.com/todos'
      );
      this.todos = data;
    }
  }
};
</script>

<template>
  <ul>
    <li v-for="todo in todos" :key="todo.id">
      {{ todo.title }}
    </li>
  </ul>
</template>
```

데이터 패치는 `created` 옵션을 사용하는 게 필수였다.

Composition API

```html
<script setup>
import { ref } from 'vue';
import axios from 'axios';

const todos = ref([]);

async function getTodos() {
  const { data } = await axios.get(
    'https://jsonplaceholder.typicode.com/todos'
  );
  todos.value = data;
}

getTodos();
</script>

<template>
  <ul>
    <li v-for="todo in todos" :key="todo.id">
      {{ todo.title }}
    </li>
  </ul>
</template>
```

더이상 `created` 를 사용하지 않는다. 이렇게 작성하면, 컴포넌트 생성 시 딱 한번 데이터패치를 실행한다.

즉, 사용빈도가 낮은 `mounted` 를 배울 게 아니라면 라이프사이클이라는 개념을 초급 단계에서 학습하지 않아도 된다.

# 6. props

Options API

```html
<script>
import ChildComp from './ChildComp.vue'

export default {
  data() {
    return {
      greeting: 'Hello from parent'
    };
  },
  components: {
    ChildComp
  }
};
</script>

<template>
  <ChildComp :msg="greeting"/>
</template>
```

```html
<script>
export default {
  props: ['msg'],
};
</script>

<template>
  <h2>{{ msg || 'No props passed yet' }}</h2>
</template>
```

props 을 받는 자식 컴포넌트는 Type Check 가 강제되지 않는다. 배열로 써도 상관없다.

Composition API

```html
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const greeting = ref('Hello from parent')
</script>

<template>
  <ChildComp :msg="greeting"/>
</template>
```

```html
<script setup>
const props = defineProps({
  msg: String
})
</script>

<template>
  <h2>{{ msg || 'No props passed yet' }}</h2>
</template>
```

`defineProps` 함수를 반드시 사용해야 한다. 매개변수로 객체를 필요로 하고, props 의 타입을 키 밸류 형식으로 명시해야 한다. 즉, Type Check 를 강제한다.

`defineProps` 는 컴파일타임 매크로이기 때문에, `import` 하지 않는다.

# 7. emit

Options API

```html
<script>
import ChildComp from '@/components/ChildComp.vue'

export default {
  data() {
    return {
      childMsg: 'No child msg yet'
    };
  },
  methods: {
    btnTest(arg) {
      this.childMsg = arg;
    }
  },
  components: {
    ChildComp
  }
};
</script>

<template>
  <ChildComp @btnTest="btnTest"/>
  <p>{{ childMsg }}</p>
</template>
```

```html
<script>
export default {
  methods: {
    btnTest() {
      this.$emit('btnTest', 'hello from child');
    }
  }
};
</script>

<template>
  <button @click="btnTest">click</button>
</template>
```

`this.$emit` 을 사용한다.

Composition API

```html
<script setup>
import { ref } from 'vue'
import ChildComp from '@/components/ChildComp.vue'

const childMsg = ref('No child msg yet')

function btnTest(arg) {
  childMsg.value = arg;
}
</script>

<template>
  <ChildComp @btnTest="btnTest"/>
  <p>{{ childMsg }}</p>
</template>
```

```html
<script setup>
const emit = defineEmits(['btnTest'])

function btnTest() {
  emit('btnTest', 'hello from child');
}
</script>

<template>
  <button @click="btnTest">click</button>
</template>
```

`defineEmits` 함수를 사용한다. 마찬가지로 컴파일타임 매크로이므로 `import` 하지 않는다.

`emit` 을 정의하는 절차가 추가되었으며, 정의는 자식 컴포넌트에서 한다.

사용 시엔 `this.$emit` 이 아니라 `emit` 이다.

# 8. Vuex  vs  Pinia

Vue 3 부터는, Options API, Composition API 사용 여부와 상관없이 Pinia 사용을 권장한다.

Pinia 가 발전한 점

1. 여러개의 store
    - `store/index.js` 라는 하나의 store 만 존재했던 Vuex 에서 발전해, 여러 개의 store 생성 가능.
    - 예를 들면, `stores/user.js` , `stores/article.js` 식으로 여러 개 store 를 생성해 관리한다.
    - 따라서 디렉터리 이름도 단수 `store/` 가 아니라, 복수 `stores/` 다.
2. 문법이 훨씬 쉬워졌다.

Vuex

```jsx
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    name: "jony",
    age: 40,
  },
  getters: {},
  mutations: {
    CHANGE_NAME(state, newName) {
      state.name = newName;
    },
    CHANGE_AGE(state, newAge) {
      state.age = newAge;
    },
  },
  actions: {
    changeName(context, newName) {
      context.commit("CHANGE_NAME", newName);
    },
    changeAge(context, newAge) {
      context.commit("CHANGE_AGE", newAge);
    },
  },
});
```

`state` , `getters` , `mutations` , `actions` 각각의 옵션으로 나뉘어져 있고, 각 옵션마다 복잡한 문법으로 이루어져있다.

사용하는 방법도 어려웠다.

```html
<script>
import { mapState, mapActions } from 'vuex';

export default {
  computed: {
    ...mapState(['name', 'age'])
  },
  methods: {
    ...mapActions(['changeName', 'changeAge'])
  }
};
</script>
```

이렇게 spread 를 사용해서 가져왔어야 했다. 즉, spread 를 모를 경우, 학습이 필요했다.

Pinia

```jsx
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useUserStore = defineStore('userStore', () => {
  const name = ref('jony')
  const age = ref(40)

  function changeName(newName) {
    name.value = newName;
  }

  function changeAge(newAge) {
    age.value = newAge;
  }

  return { name, age, changeName, changeAge }
}
```

`userStore` 를 정의했다.

훨씬 간결해졌다.

`state` ⇒ `ref` 사용

`mutations` ⇒ deprecated

`actions` ⇒ `function`

`getters` ⇒ `computed`

사용하려면 `return` 해줘야 한다.

컴포넌트에서 사용은 다음과 같다.

```html
<script setup>
import { useUserStore } from '@/stores/userStore.js'

const userStore = useUserStore()
</script>

<template>
  <p>{{ userStore.name }}</p>
  <p>{{ userStore.age }}</p>
  <button @click="userStore.changeName('newName')">Change Name</button>
  <button @click="userStore.changeAge(50)">Change Age</button>
</template>
```

`useUserStore` 를 가져온 후, 리턴값 `userStore` 객체를 받아주고 편하게 사용하면 된다.

자주 혼란을 일으키는 개념인데, `<script setup>` 안에서 사용할 땐 `ref` 로 선언한 상태와 다르게 `.value` 를 붙이지 않는다. 

잠시 Composable 개념을 설명하겠다.

Composition API 에서 새로 생긴 개념이며, React Hooks 에서 착안한 개념이다. 상태를 포함한 함수이며, `use-` 접두사를 붙인다.

특히 Pinia, Vue Router 를 사용할 때 자주 보게 된다.

`useUserStore` 는 처음 등장한 Composable 이며, `ref` 로 선언한 상태를 포함하고 있다.

Composable 을 가져와 사용할 땐, 위에 제시된 코드처럼 한번 실행해 객체를 받아온 후 사용한다.

# 9. Vue Router

Options API

```html
<template>
  <h1>HomeView</h1>
  <button @click="goToBoardRoute">게시판으로 이동하자!</button>
</template>

<script>
export default {
  methods: {
    goToBoardRoute() {
      this.$router.push({ name: "board" });
    },
  },
};
</script>
```

별도의 `import` 구문 없이 `this.$router` 를 사용한다.

Composition API

```html
<template>
  <h1>HomeView</h1>
  <button @click="goToBoardRoute">게시판으로 이동하자!</button>
</template>

<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

function goToBoardRoute() {
  router.push({ name: "board" });
}
</script>
```

`useRoute` , `useRouter` Composable 을 사용하므로, `vue-router` 에서 `import` 한 후, 객체 `route` , `router` 로 만들어 사용한다.

`this.$router` 가 아니라 `router` , `this.$route` 가 아니라 `route` 이다.