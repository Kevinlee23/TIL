### 实现带搜索栏的 tabs

```vue
<template>
  <div class="relative">
    <div class="inner-container overflow-x-hidden">
      <div class="tabs-content" :class="[transformStyle]">
        <!-- 实现tabs -->
        <div class="tabs-left">...</div>

        <!-- 实现搜索input -->
        <div class="tabs-right">...</div>
      </div>
    </div>

    <!-- 搜索icon -->
    <span v-show="!isHidden" ref="iconEl" class="search-icon" @click.stop="handleHidden">
      <icon>...</icon>
    </span>
  </div>
</template>
<script setup>
import { ref, computed } from "vue";

// tab栏是否隐藏 (打开搜索栏)
const isHidden = ref(false);
const transformStyle = computed(() => {
  return isHidden.value ? "-translated-x-[50%]" : "translate-x-0";
});
const handleHidden = () => {
  isHidden.value = true; 
};

const iconEl = ref(null);
onClickOutside(iconEl, () => {
  isHidden.value = false;
});
</script>

<style lang="scss">
.tabs-content {
  @apply flex w-[200%] duration-300;

  .tabs-left {
    @apply w-[50%] px-[40px];
  }

  .tabs-right {
    @apply w-[50%] px-[10px];
  }
}

.search-icon {
  @apply absolute top-0 right-[10px] cursor-pointer;
}
</style>
```
