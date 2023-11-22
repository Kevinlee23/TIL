### pinia 基本示例

pinia 定义 store 的三个重要部分:

- state 状态
- getters 计算状态
- actions 定义业务逻辑

你可以像使用组合式函数 (hook) 一样定义/使用 store 而不需要像 vuex 一样作为插件注册 store 实例

#### 示例
```javascript
import { defineStore } from "pinia";

// Pinia 将使用定义时注册的名字来连接到 devtools
export const useTodosStore = defineStore("todos", {
  state: () => ({
    /** @type {{ text: string, id: number, isFinished: boolean}[]} */
    todos: [],
    /** @type {'all' | 'finished' | 'unfinished'} */
    filter: "all",
    /** @type number */
    nextId: 0,
  }),
  getters: {
    finishedTodos(state) {
      return state.todos.filter((todo) => todo.isFinished);
    },
    unfinishedTodos(state) {
      return state.todos.filter((todo) => !todo.isFinished);
    },
    /**
     * @returns {{ text: string, id: number, isFinished: boolean }[]}
     */
    filteredTodos(state) {
      if (this.filter === "finised") {
        return this.finishedTodos;
      } else if (this.filter === "unfinished") {
        return this.unfinishedTodos;
      }

      return this.todos;
    },
  },
  actions: {
    // 任何数量的参数, 返回一个 Promise 或者不返回
    addTodo(text) {
      // 和 vuex 不同的是, 可以直接改变状态
      this.todos.push({ text, id: this.nextId++, isFinished: false });
    },
  },
});
```

#### 挂载

```javascript
// @/store/index.js
import { createPinia } from "pinia";

const store = createPinia();

export default store;

// main.js
import root from "./App.vue";
import store from "./store";
import router from "./router";

instance = createApp(root);
instance.use(router).use(store);
instance.mount(container.querySelector("#app"));
```

#### 使用

```javascript
import { useTodosStore } from "@/stores/todos";

// x. 会破坏响应式
const store = useTodosStore();
const { todos, nextId } = store;

// √. 直接使用 state 或者 action
const store = useTodosStore();
const todos = store.todos;
const finishedTodos = store.finishedTodos;
const unFinishedTodos = store.unfinishedTodos;
store.addTodo("Write one TIL");
```
