### 全局挂载

`main.js`

```js
import { createApp } from "vue";
import App from "./App.vue";

const app = createApp(App);
app.config.globalProperties.name = "麓下雪";
```

`a.vue`

```vue
<script setup>
const { proxy, appContext } = getCurrentInstance();
const global = appContext.config.globalProperties;

console.log(global.name); // 麓下雪
</script>
```
