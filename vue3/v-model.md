### v-model

vue3 支持多个 v-model，修改使用 $emit('update:name')

`父组件`

```vue
<template>
  <child v-model:name="name" v-model:age="age"></child>
</template>
<script setup>
const name = ref("luxiaxue");
const age = ref(26);
</script>
```

`Child.vue`

```vue
<script setup>
const props = defineProps({ name: String, age: Number });
const emit = defineEmits(['update:name', 'update:age']);

// 修改入参 props.name
const handleNameChange = () => {
  emit('update:name', 'lixiuxian')
}
<scirpt>
```

也可以用 computed 中的 set 方法来简化:

`Child.vue`

```vue
<script setup>
const props = defineProps({ name: String, age: Number });
const emit = defineEmits(["update:name", "update:age"]);

const model = computed({
  get() {
    return props.name;
  },
  set(val) {
    emit("update:name", val);
  },
});

const handleNameChange = () => {
  model.value = "lixiuxian";
};
</script>
```

使用 vue-use 库中的方法会更加简单:

```vue
<script setup>
import { useVModel, useVModel } from "@vueuse/core";

const props = definePrpos({
  foo: String,
  bar: Number,
  zbb: Boolean,
});

const fooModel = useVModel(props, 'foo' emit);
const { barModel, zbbModel } = useVmodels(props, emit);

console.log(fooModel.value, barModel) // props.foo, props.bar
zbbModel.value = false // emit('updata:zbb', false)
</script>
```
