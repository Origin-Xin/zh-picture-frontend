<template>
  <div class="no-auth-page">
    <a-result status="403" title="403" sub-title="抱歉，您没有访问此页面的权限">
      <template #extra>
        <a-button type="primary" @click="goToHome"> 返回首页 </a-button>
        <a-button @click="goToLogin" style="margin-left: 8px"> 重新登录 </a-button>
      </template>
    </a-result>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'

const router = useRouter()

const goToHome = () => {
  router.push('/')
}

const goToLogin = () => {
  router.push('/user/login')
}
</script>

<style scoped>
.no-auth-page {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 60vh;
}
</style>
