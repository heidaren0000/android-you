<script setup>
import { RouterLink, RouterView, useRouter } from 'vue-router'

// import { ref } from 'vue'
// 导入 v-router, 这样在 事件绑定中可以使用
const router = useRouter()
</script>

<template>
  <mdui-navigation-rail value="Phone" alignment="center" style="position: fixed">
    <mdui-navigation-rail-item @click="router.push('/')" icon="phonelink--outlined" value="Phone"
      >我的手机</mdui-navigation-rail-item
    >
    <mdui-navigation-rail-item @click="router.push('/about')" icon="info--outlined" value="About"
      >关于</mdui-navigation-rail-item
    >
  </mdui-navigation-rail>
  <RouterView />
</template>

<style scoped></style>
