<template>
  <div id="GlobalHeader">
    <div class="header-container">
      <!-- Logo区域 -->
      <div class="logo-section">
        <RouterLink to="/" class="logo-link">
          <div class="logo-wrapper">
            <img class="logo" src="../assets/logo.ico" alt="logo" />
            <div class="brand-text">
              <div class="title">
                <span class="title-gradient">智能云图库</span>
              </div>
            </div>
          </div>
        </RouterLink>
      </div>

      <!-- 中间空白区域 -->
      <div class="center-spacer"></div>

      <!-- 右侧操作区域 -->
      <div class="action-section">
        <div class="decoration-icons">
          <span class="decoration-icon">🍔</span>
          <span class="decoration-icon">🍟</span>
          <span class="decoration-icon">🌭</span>
        </div>
        
        <!-- 未登录状态 -->
        <template v-if="!isLoggedIn">
          <a-button
            type="primary"
            size="small"
            @click="$router.push('/user/login')"
            class="login-btn"
          >
            登录
          </a-button>
        </template>
        
        <!-- 已登录状态 -->
        <template v-else>
          <a-dropdown placement="bottomRight">
            <div class="user-info">
              <a-avatar 
                :src="loginUserStore.loginUser.userAvatar" 
                :size="32"
                class="user-avatar"
              >
                {{ loginUserStore.loginUser.userName?.charAt(0) }}
              </a-avatar>
              <span class="username">{{ loginUserStore.loginUser.userName }}</span>
              <DownOutlined class="dropdown-icon" />
            </div>
            <template #overlay>
              <a-menu>
                <a-menu-item @click="goToProfile">
                  <UserOutlined />
                  个人中心
                </a-menu-item>
                <a-menu-divider />
                <a-menu-item @click="doLogout">
                  <LogoutOutlined />
                  退出登录
                </a-menu-item>
              </a-menu>
            </template>
          </a-dropdown>
        </template>
      </div>
    </div>
  </div>
</template>

<style scoped>
#GlobalHeader {
  background: linear-gradient(
    135deg,
    rgba(24, 144, 255, 0.08) 0%,
    rgba(135, 206, 235, 0.12) 25%,
    rgba(255, 255, 255, 0.15) 50%,
    rgba(135, 206, 235, 0.12) 75%,
    rgba(255, 250, 240, 0.18) 100%
  );
  backdrop-filter: blur(40px) saturate(1.8) brightness(1.1);
  -webkit-backdrop-filter: blur(40px) saturate(1.8) brightness(1.1);
  box-shadow:
    0 8px 32px rgba(24, 144, 255, 0.12),
    0 4px 16px rgba(0, 0, 0, 0.08),
    0 2px 8px rgba(255, 255, 255, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    inset 0 -1px 0 rgba(24, 144, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 16px;
  margin: 12px 16px 8px 16px;
  position: sticky;
  top: 12px;
  z-index: 1000;
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.header-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 64px;
  padding: 0 24px;
  max-width: 100%;
  margin: 0;
  position: relative;
}

/* Logo区域 */
.logo-section {
  flex: 0 0 auto;
}

.logo-link {
  text-decoration: none;
  color: inherit;
}

.logo-wrapper {
  display: flex;
  align-items: center;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.logo-wrapper:hover {
  transform: translateY(-1px);
}

.logo {
  height: 32px;
  width: 32px;
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.logo:hover {
  transform: rotate(5deg) scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.brand-text {
  margin-left: 10px;
}

.title {
  font-size: 18px;
  font-weight: 600;
  margin: 0;
  line-height: 1.2;
}

.title-gradient {
  background: linear-gradient(45deg, #1890ff 20%, #40a9ff 50%, #597ef7 80%);
  background-size: 200% 200%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 1px 2px rgba(24, 144, 255, 0.2);
  animation: shimmer 4s ease-in-out infinite;
  white-space: nowrap;
}

@keyframes shimmer {
  0%,
  100% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
}

/* 中间空白区域 */
.center-spacer {
  flex: 1;
}

/* 右侧操作区域 */
.action-section {
  flex: 0 0 auto;
  display: flex;
  align-items: center;
  gap: 12px;
  max-width: 200px;
}

.decoration-icons {
  display: flex;
  gap: 8px;
}

.decoration-icon {
  font-size: 18px;
  opacity: 0.8;
  animation: bounce 3s ease-in-out infinite;
}

.decoration-icon:nth-child(1) {
  animation-delay: 0s;
}

.decoration-icon:nth-child(2) {
  animation-delay: 0.5s;
}

.decoration-icon:nth-child(3) {
  animation-delay: 1s;
}

@keyframes bounce {
  0%,
  100% {
    transform: translateY(0) scale(1);
    opacity: 0.8;
  }
  50% {
    transform: translateY(-3px) scale(1.05);
    opacity: 1;
  }
}

/* 登录按钮 */
.login-btn {
  height: 32px !important;
  font-size: 13px !important;
  font-weight: 500 !important;
  border-radius: 8px !important;
  background: linear-gradient(135deg, #1890ff 0%, #40a9ff 100%) !important;
  border: none !important;
  box-shadow: 0 2px 8px rgba(24, 144, 255, 0.3) !important;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1) !important;
  flex-shrink: 0;
}

.login-btn:hover {
  background: linear-gradient(135deg, #096dd9 0%, #1890ff 100%) !important;
  transform: translateY(-1px) !important;
  box-shadow: 0 4px 12px rgba(24, 144, 255, 0.4) !important;
}

.login-btn:active {
  transform: translateY(0) !important;
}

/* 用户信息区域 */
.user-info {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 4px 8px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.user-info:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.user-avatar {
  flex-shrink: 0;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.username {
  font-size: 13px;
  font-weight: 500;
  color: #1890ff;
  white-space: nowrap;
  max-width: 80px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.dropdown-icon {
  font-size: 10px;
  color: #1890ff;
  transition: transform 0.3s ease;
}

.user-info:hover .dropdown-icon {
  transform: rotate(180deg);
}

/* 响应式设计 */
@media (max-width: 768px) {
  #GlobalHeader {
    margin: 6px 8px 2px 8px;
    top: 6px;
  }

  .header-container {
    padding: 0 16px;
    height: 48px;
  }

  .title {
    font-size: 16px;
  }

  .logo {
    height: 28px;
    width: 28px;
  }

  .decoration-icons {
    display: none;
  }

  .action-section {
    gap: 8px;
    max-width: 80px;
  }

  .login-btn {
    height: 28px !important;
    font-size: 12px !important;
    padding: 0 8px !important;
  }

  .user-info {
    padding: 2px 6px;
    gap: 6px;
  }

  .username {
    font-size: 12px;
    max-width: 60px;
  }

  .user-avatar {
    width: 24px !important;
    height: 24px !important;
  }
}

@media (max-width: 480px) {
  #GlobalHeader {
    margin: 3px 4px 1px 4px;
    top: 3px;
  }

  .header-container {
    padding: 0 12px;
  }

  .title {
    font-size: 14px;
  }

  .logo {
    height: 24px;
    width: 24px;
  }

  .brand-text {
    margin-left: 6px;
  }

  .action-section {
    gap: 6px;
    max-width: 60px;
  }

  .login-btn {
    height: 24px !important;
    font-size: 11px !important;
    padding: 0 6px !important;
    min-width: 40px !important;
  }

  .user-info {
    padding: 1px 4px;
    gap: 4px;
  }

  .username {
    font-size: 11px;
    max-width: 40px;
  }

  .user-avatar {
    width: 20px !important;
    height: 20px !important;
  }
}
</style>
<script lang="ts" setup>
import { computed } from 'vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore'
import { message } from 'ant-design-vue'
import { 
  UserOutlined, 
  LogoutOutlined, 
  DownOutlined 
} from '@ant-design/icons-vue'
import ACCESS_ENUM from '@/access/accessEnum'

const router = useRouter()
const loginUserStore = useLoginUserStore()

// 判断是否已登录
const isLoggedIn = computed(() => {
  return loginUserStore.loginUser && 
         loginUserStore.loginUser.id && 
         loginUserStore.loginUser.userRole !== ACCESS_ENUM.NOT_LOGIN
})

// 跳转到个人中心
const goToProfile = () => {
  router.push('/user/profile')
}

// 退出登录
const doLogout = async () => {
  try {
    // 清除登录状态
    loginUserStore.setLoginUser({
      userName: '未登录',
      userRole: ACCESS_ENUM.NOT_LOGIN,
    })
    message.success('已退出登录')
    // 跳转到首页
    router.push('/')
  } catch (error) {
    message.error('退出登录失败')
  }
}
</script>
