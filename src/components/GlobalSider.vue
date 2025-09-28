<template>
  <div id="globalSider">
    <a-layout-sider
      v-if="loginUserStore.loginUser.id"
      class="sider"
      width="180"
      breakpoint="lg"
      :collapsed-width="64"
      :collapsed="state.collapsed"
      :trigger="null"
      @mouseenter="handleMouseEnter"
      @mouseleave="handleMouseLeave"
    >
      <!-- 用户信息区域 -->
      <div class="user-info-section">
        <div v-if="!state.collapsed" class="user-info-expanded">
          <div class="user-basic-info">
            <a-avatar :src="loginUserStore.loginUser.userAvatar" :size="40" />
            <span class="username">{{ loginUserStore.loginUser.userName ?? '无名' }}</span>
          </div>
        </div>
        <div v-else class="user-info-collapsed">
          <a-dropdown placement="rightTop">
            <a-avatar
              :src="loginUserStore.loginUser.userAvatar"
              :size="48"
              class="user-avatar-collapsed"
            />
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
        </div>
      </div>

      <!-- 主菜单 -->
      <a-menu
        v-model:openKeys="state.openKeys"
        v-model:selectedKeys="state.selectedKeys"
        mode="inline"
        theme="light"
        :inline-collapsed="state.collapsed"
        :items="menuItems"
        @click="doMenuClick"
        class="main-menu"
      />
    </a-layout-sider>
  </div>
</template>

<script lang="ts" setup>
import { computed, h, reactive, ref, watch, watchEffect } from 'vue'
import {
  // 使用双色图标和更贴合文字的图标
  HomeTwoTone,
  FolderTwoTone,
  DatabaseTwoTone,
  TeamOutlined,
  SettingTwoTone,
  InfoCircleTwoTone,
  UserOutlined,
  LogoutOutlined,
  CloudServerOutlined,
} from '@ant-design/icons-vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { listMyTeamSpaceUsingPost } from '@/api/spaceUserController.ts'
import { message } from 'ant-design-vue'
import checkAccess from '@/access/checkAccess'
import routerInstance from '@/router'
import { userLogoutUsingPost } from '@/api/userController.ts'

const loginUserStore = useLoginUserStore()

// 主菜单项 - 使用双色图标和贴合文字的图标
// 自定义 SVG 图标组件 - 图片创建图标
const CustomPictureIcon = () =>
  h(
    'svg',
    {
      viewBox: '0 0 1024 1024',
      width: '1em',
      height: '1em',
      fill: 'currentColor',
      style: { verticalAlign: '-0.15em' }, // 更好地对齐文字
    },
    [
      h('path', {
        d: 'M928 160H96c-17.7 0-32 14.3-32 32v640c0 17.7 14.3 32 32 32h832c17.7 0 32-14.3 32-32V192c0-17.7-14.3-32-32-32z',
        fill: '#1890ff',
        opacity: 0.15,
      }),
      h('path', {
        d: 'M704 288c0 17.7-14.3 32-32 32H352c-17.7 0-32-14.3-32-32s14.3-32 32-32h320c17.7 0 32 14.3 32 32zm0 144c0 17.7-14.3 32-32 32H352c-17.7 0-32-14.3-32-32s14.3-32 32-32h320c17.7 0 32 14.3 32 32z',
        fill: '#1890ff',
      }),
      h('path', {
        d: 'M928 160H96c-17.7 0-32 14.3-32 32v640c0 17.7 14.3 32 32 32h832c17.7 0 32-14.3 32-32V192c0-17.7-14.3-32-32-32zM864 832H160V224h704v608z',
        fill: 'currentColor',
      }),
    ],
  )

// 自定义用户空间图标
const CustomUserSpaceIcon = () =>
  h(
    'svg',
    {
      viewBox: '0 0 1024 1024',
      width: '1em',
      height: '1em',
      fill: 'currentColor',
      style: { verticalAlign: '-0.15em' },
    },
    [
      h('path', {
        d: 'M512 64C264.6 64 64 264.6 64 512s200.6 448 448 448 448-200.6 448-448S759.4 64 512 64z',
        fill: '#52c41a',
        opacity: 0.15,
      }),
      h('path', {
        d: 'M512 288c123.7 0 224 100.3 224 224S635.7 736 512 736 288 635.7 288 512s100.3-224 224-224z',
        fill: '#52c41a',
      }),
      h('path', {
        d: 'M512 64C264.6 64 64 264.6 64 512s200.6 448 448 448 448-200.6 448-448S759.4 64 512 64zm0 820c-205.4 0-372-166.6-372-372s166.6-372 372-372 372 166.6 372 372-166.6 372-372 372z',
        fill: 'currentColor',
      }),
    ],
  )

// 自定义空间分析图标
const CustomAnalyzeIcon = () =>
  h(
    'svg',
    {
      viewBox: '0 0 1024 1024',
      width: '1em',
      height: '1em',
      fill: 'currentColor',
      style: { verticalAlign: '-0.15em' },
    },
    [
      h('path', {
        d: 'M928 832H96c-17.7 0-32-14.3-32-32V192c0-17.7 14.3-32 32-32h832c17.7 0 32 14.3 32 32v608c0 17.7-14.3 32-32 32z',
        fill: '#722ed1',
        opacity: 0.15,
      }),
      h('path', {
        d: 'M224 768h80V416h-80v352zm160-224h80v224h-80V544zm160-128h80v352h-80V416zm160 64h80v288h-80V480z',
        fill: '#722ed1',
      }),
      h('path', {
        d: 'M928 160H96c-17.7 0-32 14.3-32 32v640c0 17.7 14.3 32 32 32h832c17.7 0 32-14.3 32-32V192c0-17.7-14.3-32-32-32zM864 800H160V224h704v576z',
        fill: 'currentColor',
      }),
    ],
  )

const mainMenuItems: MenuItem[] = [
  {
    key: '/user/profile',
    icon: () => h(UserOutlined, { style: { color: '#1890ff' } }),
    label: '个人中心',
    title: '个人中心',
  },
  {
    key: '/',
    icon: () => h(HomeTwoTone, { twoToneColor: '#52c41a' }),
    label: '主页',
    title: '主页',
  },
  {
    key: '/my_space',
    icon: () => CustomUserSpaceIcon(),
    label: '我的空间',
    title: '我的空间',
  },
  {
    key: '/add_picture',
    icon: () => CustomPictureIcon(),
    label: '创建图片',
    title: '创建图片',
  },
  {
    key: '/add_space',
    icon: () => h(FolderTwoTone, { twoToneColor: '#1890ff' }),
    label: '创建空间',
    title: '创建空间',
  },
  {
    key: '/space_analyze',
    icon: () => CustomAnalyzeIcon(),
    label: '空间分析',
    title: '空间分析',
  },
  {
    key: 'admin-menu',
    icon: () => h(SettingTwoTone, { twoToneColor: '#eb2f96' }),
    label: '系统管理',
    title: '系统管理',
    children: [
      {
        key: '/admin/pictureManage',
        icon: () => h(DatabaseTwoTone, { twoToneColor: '#13c2c2' }),
        label: '图片管理',
        title: '图片管理',
      },
      {
        key: '/admin/userManage',
        icon: () => h(TeamOutlined, { style: { color: '#fa8c16' } }),
        label: '用户管理',
        title: '用户管理',
      },
      {
        key: '/admin/spaceManage',
        icon: () => h(CloudServerOutlined, { style: { color: '#52c41a' } }),
        label: '空间管理',
        title: '空间管理',
      },
    ],
  },
  {
    key: '/bout',
    icon: () => h(InfoCircleTwoTone, { twoToneColor: '#1890ff' }),
    label: '关于',
    title: '关于',
  },
  {
    key: 'logout',
    icon: () => h(LogoutOutlined, { style: { color: '#ff4d4f' } }),
    label: '退出登录',
    title: '退出登录',
  },
]

// 定义菜单项类型
interface MenuItem {
  key: string
  icon?: () => unknown
  label: string
  title: string
  children?: MenuItem[]
}

// 根据权限过滤菜单项
const filterMenuByAccess = (menuItems: MenuItem[]): MenuItem[] => {
  return menuItems.filter((menuItem) => {
    // 根据菜单key找到对应的路由
    const route = routerInstance.getRoutes().find((route) => route.path === menuItem.key)

    // 如果路由设置了hideInMenu，则隐藏菜单项
    if (route?.meta?.hideInMenu) {
      return false
    }

    // 检查用户是否有访问权限
    const needAccess = route?.meta?.access as string
    const hasAccess = checkAccess(loginUserStore.loginUser, needAccess)

    if (!hasAccess) {
      return false
    }

    // 如果有子菜单，递归过滤
    if (menuItem.children && menuItem.children.length > 0) {
      menuItem.children = filterMenuByAccess(menuItem.children)
      // 如果过滤后子菜单为空，则隐藏父菜单
      return menuItem.children.length > 0
    }

    return true
  })
}

const teamSpaceList = ref<API.SpaceUserVO[]>([])

// 菜单列表 - 应用权限过滤逻辑
const menuItems = computed(() => {
  // 应用权限过滤
  return filterMenuByAccess(mainMenuItems)
})

// 加载团队空间列表
const fetchTeamSpaceList = async () => {
  const res = await listMyTeamSpaceUsingPost()
  if (res.data.code === 0 && res.data.data) {
    teamSpaceList.value = res.data.data
  } else {
    message.error('加载我的团队空间失败，' + res.data.message)
  }
}

/**
 * 监听变量，改变时触发数据的重新加载
 */
watchEffect(() => {
  // 登录才加载
  if (loginUserStore.loginUser.id) {
    fetchTeamSpaceList()
  }
})

// 菜单状态管理
const state = reactive({
  collapsed: true, // 默认折叠状态
  selectedKeys: [routerInstance.currentRoute.value.path],
  openKeys: ['admin-menu'],
  preOpenKeys: ['admin-menu'],
})

// 监听路由变化，更新当前选中菜单
routerInstance.afterEach((to) => {
  state.selectedKeys = [to.path]
})

// 监听 openKeys 变化（按照官方示例）
watch(
  () => state.openKeys,
  (_val, oldVal) => {
    state.preOpenKeys = oldVal || []
  },
)

// 鼠标悬停处理函数
const handleMouseEnter = () => {
  state.collapsed = false
  state.openKeys = state.preOpenKeys
}

const handleMouseLeave = () => {
  state.collapsed = true
  state.openKeys = []
}

// 路由跳转事件
const doMenuClick = ({ key }: { key: string }) => {
  if (key === 'logout') {
    doLogout()
  } else {
    routerInstance.push(key)
  }
}

// 跳转到个人中心
const goToProfile = () => {
  routerInstance.push('/user/profile')
}

// 用户注销
const doLogout = async () => {
  const res = await userLogoutUsingPost()
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await routerInstance.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}
</script>

<style scoped>
#globalSider .ant-layout-sider {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  box-shadow: 4px 0 16px rgba(0, 0, 0, 0.08);
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  margin-top: 8px;
  margin-bottom: 28px;
  margin-left: 8px;
  margin-right: 8px;
  min-height: calc(120vh - 180px);
  overflow: hidden;
}

/* 将悬停阴影效果修改为更细膩的边框变化 */
#globalSider .ant-layout-sider:hover {
  border: 1px solid rgba(24, 144, 255, 0.2);
  box-shadow: 6px 0 20px rgba(24, 144, 255, 0.12);
}

/* 用户信息区域样式 */
.user-info-section {
  padding: 16px 12px;
  border-bottom: 1px solid rgba(240, 240, 240, 0.6);
  background: rgba(250, 250, 250, 0.5);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  border-radius: 16px 16px 0 0;
  margin-bottom: 8px;
  transition: all 0.3s ease;
}

.user-info-expanded {
  display: flex;
  flex-direction: column;
}

.user-basic-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.username {
  font-size: 14px;
  font-weight: 500;
  color: #333;
  flex: 1;
  margin-left: 12px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.user-info-collapsed {
  display: flex;
  justify-content: center;
  align-items: center;
}

.user-avatar-collapsed {
  cursor: pointer;
  transition: all 0.3s ease;
}

.user-avatar-collapsed:hover {
  transform: scale(1.05);
}

#globalSider :deep(.ant-layout-sider-children) {
  display: flex;
  flex-direction: column;
  height: 100%;
}

#globalSider :deep(.ant-menu) {
  border-right: none;
  font-size: 13px;
  transition: all 0.3s ease;
  flex: 1;
  background: transparent;
  border-radius: 0 0 16px 16px;
  padding-bottom: 16px;
}

#globalSider :deep(.ant-menu-item) {
  height: 36px;
  line-height: 36px;
  margin: 2px 6px;
  border-radius: 4px;
  overflow: hidden;
  padding: 0 12px !important;
  transition: all 0.3s ease;
}

#globalSider :deep(.ant-menu-item-icon) {
  font-size: 14px;
  margin-inline-end: 8px;
  transition: all 0.3s ease;
}

#globalSider :deep(.ant-menu-item-selected) {
  background-color: rgba(230, 247, 255, 0.9);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  color: #1890ff;
  font-weight: 600;
}

#globalSider :deep(.ant-menu-item-selected::after) {
  border-right: 3px solid #1890ff;
}

#globalSider :deep(.ant-menu-item:hover) {
  background-color: rgba(245, 245, 245, 0.8);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
}

/* 折叠状态下的样式 */
#globalSider :deep(.ant-menu-inline-collapsed) {
  width: 64px;
}

#globalSider :deep(.ant-menu-inline-collapsed .ant-menu-item) {
  text-align: center;
  padding: 0 !important;
  display: flex;
  align-items: center;
  justify-content: center;
}

#globalSider :deep(.ant-menu-inline-collapsed .ant-menu-item-icon) {
  font-size: 16px;
  margin-inline-end: 0;
}

/* 折叠时隐藏文字，只显示图标 */
#globalSider :deep(.ant-menu-inline-collapsed .ant-menu-title-content) {
  display: none;
}

/* 在折叠状态下显示提示条 */
#globalSider .ant-layout-sider-collapsed::before {
  content: '';
  position: absolute;
  top: 50%;
  right: -2px;
  transform: translateY(-50%);
  width: 4px;
  height: 40px;
  background: linear-gradient(to bottom, transparent, #1890ff, transparent);
  border-radius: 2px;
  opacity: 0;
  transition: all 0.3s ease;
}

#globalSider .ant-layout-sider-collapsed:hover::before {
  opacity: 1;
}

/* 响应式高度调整 - 与内容区域保持一致 */
@media (max-width: 768px) {
  #globalSider .ant-layout-sider {
    min-height: calc(100vh - 140px);
  }
}

@media (max-width: 480px) {
  #globalSider .ant-layout-sider {
    min-height: calc(100vh - 120px);
  }
}
</style>
