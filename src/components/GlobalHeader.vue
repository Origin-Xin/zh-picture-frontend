<template>
  <div id="GlobalHeader">
    <a-row :wrap="false">
      <a-col flex="200px">
        <RouterLink to="/">
          <div class="title-bar">
            <img class="logo" src="../assets/logo.ico" alt="logo" />
            <div class="title">智能云图库</div>
          </div>
        </RouterLink>
      </a-col>
      <a-col flex="auto">
        <a-menu
          v-model:selectedKeys="current"
          mode="horizontal"
          :items="items"
          @click="doMenuClick"
        />
      </a-col>
      <a-col flex="120px">
        <div class="user-login-status">
          <div v-if="loginUserStore.loginUser.id">
            <a-dropdown>
              <ASpace>
                <a-avatar :src="loginUserStore.loginUser.userAvatar" />
                {{ loginUserStore.loginUser.userName ?? '无名' }}
              </ASpace>
              <template #overlay>
                <a-menu>
                  <a-menu-item>
                    <router-link to="/user/profile">
                      <UserOutlined />
                      个人中心
                    </router-link>
                  </a-menu-item>
                  <a-menu-item>
                    <router-link to="/my_space">
                      <UserOutlined />
                      我的空间
                    </router-link>
                  </a-menu-item>
                  <a-menu-divider />
                  <a-menu-item @click="doLogout">
                    <LogoutOutlined />
                    退出登录
                  </a-menu-item>
                </a-menu>
              </template>
            </a-dropdown>
          </div>
          <div v-else>
            <a-button type="primary" href="/user/login">登录</a-button>
          </div>
        </div>
      </a-col>
    </a-row>
  </div>
</template>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: black;
  font-size: 18px;
  margin-left: 16px;
}

.logo {
  height: 48px;
}
</style>
<script lang="ts" setup>
import { computed, h, ref } from 'vue'
import {
  MailOutlined,
  AlipayCircleOutlined,
  LogoutOutlined,
  RadarChartOutlined,
  UserOutlined,
} from '@ant-design/icons-vue'
import type { MenuProps } from 'ant-design-vue'
import { message } from 'ant-design-vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { userLogoutUsingPost } from '@/api/userController.ts'
import checkAccess from '@/access/checkAccess'
import routerInstance from '@/router'
const loginUserStore = useLoginUserStore()
const router = useRouter()

const originItems = [
  {
    key: '/',
    icon: () => h(MailOutlined),
    label: '主页',
    title: '主页',
  },
  {
    key: '/bout',
    icon: () => h(AlipayCircleOutlined),
    label: '关于',
    title: '关于',
  },
  {
    key: '/add_picture',
    label: '创建图片',
    title: '创建图片',
  },
  {
    key: '/add_space',
    label: '创建空间',
    title: '创建空间',
  },
  {
    key: '/space_analyze',
    label: '空间分析',
    title: '空间分析',
  },
  {
    key: '/admin/pictureManage',
    label: '图片管理',
    title: '图片管理',
  },
  {
    key: '/admin/userManage',
    icon: () => h(RadarChartOutlined),
    label: '用户管理',
    title: '用户管理',
  },
  {
    key: '/admin/spaceManage',
    label: '空间管理',
    title: '空间管理',
  },
]

// 声明扩展的 meta 类型
interface ExtendedMeta {
  access?: string
  hideInMenu?: boolean
}

// 菜单项到路由项的转换
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const menuToRouteItem = (menu: any) => {
  // 根据菜单key找到对应的路由配置
  const route = routerInstance.getRoutes().find((route) => route.path === menu?.key)
  return route || { meta: {} as ExtendedMeta }
}

// 根据权限过滤菜单项
const filterMenus = (menus = [] as MenuProps['items']) => {
  return menus?.filter((menu) => {
    // 将菜单转换为路由项
    const routeItem = menuToRouteItem(menu)
    const meta = routeItem.meta as ExtendedMeta

    // 如果设置了隐藏菜单，则不显示
    if (meta?.hideInMenu) {
      return false
    }

    // 根据权限过滤菜单，有权限则返回 true，则保留该菜单
    return checkAccess(loginUserStore.loginUser, meta?.access)
  })
}

//
const items = computed(() => {
  return filterMenus(originItems)
})

// 路由跳转事件
const doMenuClick = ({ key }: { key: string }) => {
  router.push({
    path: key,
  })
}

const current = ref<string[]>([])
// 监听路由变化，更新当前选中菜单
router.afterEach((to) => {
  current.value = [to.path]
})

// 用户注销
const doLogout = async () => {
  const res = await userLogoutUsingPost()
  console.log(res)
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}
</script>
