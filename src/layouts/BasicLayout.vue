<template>
  <div id="basicLayout">
    <a-layout style="min-height: 100vh">
      <a-layout-header class="header">
        <GlobalHeader />
      </a-layout-header>
      <a-layout>
        <GlobalSider class="sider" />
        <a-layout-content class="content">
          <router-view />
        </a-layout-content>
      </a-layout>
    </a-layout>
  </div>
</template>

<script setup lang="ts">
import GlobalHeader from '@/components/GlobalHeader.vue'
import GlobalSider from '@/components/GlobalSider.vue'
</script>
<style scoped>
/* 页面整体背景，增强毛玻璃对比效果 */
#basicLayout {
  background: linear-gradient(
    135deg,
    #f0f8ff 0%,
    #e6f3ff 25%,
    #fff5f5 50%,
    #f8f9fa 75%,
    #ffffff 100%
  );
  min-height: 100vh;
}

/* 导航栏头部区域 */
#basicLayout .header {
  padding-inline: 0;
  margin-bottom: 0;
  color: unset;
  background: transparent;
  position: relative;
  z-index: 10;
  height: auto;
  line-height: normal;
}

/* 主内容区域 */
#basicLayout .content {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  margin: 8px 16px 28px 16px;
  padding: 24px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  min-height: calc(100vh - 180px);
}

/* 侧边栏区域 */
#basicLayout .sider {
  background: transparent;
  border-right: none;
  box-shadow: none;
  position: relative;
  z-index: 5;
  margin-right: 0;
}

/* 主布局容器调整 */
#basicLayout .ant-layout {
  background: transparent;
}

/* 内容布局调整 */
#basicLayout .ant-layout:not(.ant-layout-has-sider) {
  background: transparent;
}

/* 菜单样式优化 */
#basicLayout :deep(.ant-menu-root) {
  border-bottom: none !important;
  border-inline-end: none !important;
  background: transparent;
}

/* 响应式优化 */
@media (max-width: 768px) {
  #basicLayout .content {
    margin: 4px 8px 16px 8px;
    padding: 16px;
    min-height: calc(100vh - 140px);
  }

  #basicLayout .sider {
    /* 移动端保持透明 */
  }
}

@media (max-width: 480px) {
  #basicLayout .content {
    margin: 2px 4px 8px 4px;
    padding: 12px;
    border-radius: 8px;
  }
}
</style>
