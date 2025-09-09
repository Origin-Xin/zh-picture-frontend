<template>
  <div id="userRegisterPage">
    <h2 class="title">Origin云图库 - 用户注册</h2>
    <div class="desc">企业级智能协同云图库</div>
    <a-form
      :model="formState"
      name="basic"
      label-align="left"
      autocomplete="off"
      @finish="handleSubmit"
    >
      <a-form-item name="userAccount" :rules="[{ required: true, message: '请输入账号' }]">
        <a-input v-model:value="formState.userAccount" placeholder="请输入账号" />
      </a-form-item>
      <a-form-item
        name="userPassword"
        :rules="[
          { required: true, message: '请输入密码' },
          { min: 8, message: '密码不能小于 8 位' },
        ]"
      >
        <a-input-password v-model:value="formState.userPassword" placeholder="请输入密码" />
      </a-form-item>
      <a-form-item
        name="checkPassword"
        :rules="[
          { required: true, message: '请输入确认密码' },
          { min: 8, message: '确认密码不能小于 8 位' },
        ]"
      >
        <a-input-password v-model:value="formState.checkPassword" placeholder="请输入确认密码" />
      </a-form-item>
      <a-form-item name="captcha" :rules="[{ required: true, message: '请输入验证码' }]">
        <div class="captcha-container">
          <a-input
            v-model:value="formState.captcha"
            placeholder="请输入验证码"
            style="flex: 1; margin-right: 8px"
          />
          <div class="captcha-image" @click="refreshCaptcha">
            <img
              v-if="captchaImage"
              :src="captchaImage"
              alt="验证码"
              title="点击刷新"
              style="cursor: pointer; height: 32px"
            />
            <a-spin v-else size="small" />
          </div>
        </div>
      </a-form-item>
      <div class="tips">
        已有账号？
        <RouterLink to="/user/login">去登录</RouterLink>
      </div>
      <a-form-item>
        <a-button type="primary" html-type="submit" style="width: 100%" :loading="submitLoading">
          注册
        </a-button>
      </a-form-item>
    </a-form>
  </div>
</template>
<script setup lang="ts">
import { reactive, ref, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { userRegisterUsingPost, captchaUsingGet } from '@/api/userController.ts'

const formState = reactive<API.UserRegisterRequest>({
  userAccount: '',
  userPassword: '',
  checkPassword: '',
  captcha: '',
})

const router = useRouter()
const captchaImage = ref<string>('')
const submitLoading = ref(false)

/**
 * 获取验证码
 */
const getCaptcha = async () => {
  try {
    const res = await captchaUsingGet({
      responseType: 'blob', // 重要：设置响应类型为 blob
    })

    // 检查响应状态
    if (res.status === 200 && res.data) {
      // 释放之前的 blob URL 以防止内存泄漏
      if (captchaImage.value) {
        URL.revokeObjectURL(captchaImage.value)
      }

      // 创建新的 blob URL
      const blob = new Blob([res.data], { type: 'image/png' })
      captchaImage.value = URL.createObjectURL(blob)
    } else {
      message.error('验证码加载失败')
    }
  } catch (error) {
    console.error('获取验证码失败:', error)
    message.error('网络错误，无法获取验证码')
  }
}

/**
 * 刷新验证码
 */
const refreshCaptcha = () => {
  // 清空当前验证码输入
  formState.captcha = ''
  // 重新获取验证码
  getCaptcha()
}

/**
 * 提交表单
 * @param values
 */
const handleSubmit = async (values: API.UserRegisterRequest) => {
  // 判断两次输入的密码是否一致
  if (formState.userPassword !== formState.checkPassword) {
    message.error('二次输入的密码不一致')
    return
  }

  if (!formState.captcha || formState.captcha.trim() === '') {
    message.error('请输入验证码')
    return
  }

  submitLoading.value = true
  try {
    // 将验证码添加到提交数据中
    const submitData: API.UserRegisterRequest = {
      ...values,
      captcha: formState.captcha,
    }

    const res = await userRegisterUsingPost(submitData)
    // 注册成功，跳转到登录页面
    if (res.data.code === 0 && res.data.data) {
      message.success('注册成功')
      router.push({
        path: '/user/login',
        replace: true,
      })
    } else {
      message.error('注册失败，' + res.data.message)
      // 注册失败时刷新验证码
      refreshCaptcha()
    }
  } catch {
    message.error('注册失败，请重试')
    // 请求失败时刷新验证码
    refreshCaptcha()
  } finally {
    submitLoading.value = false
  }
}

// 页面加载时获取验证码
onMounted(() => {
  getCaptcha()
})

// 组件销毁时清理 blob URL
onUnmounted(() => {
  if (captchaImage.value) {
    URL.revokeObjectURL(captchaImage.value)
  }
})
</script>

<style scoped>
#userRegisterPage {
  max-width: 360px;
  margin: 0 auto;
}

.title {
  text-align: center;
  margin-bottom: 16px;
}

.captcha-container {
  display: flex;
  align-items: center;
}

.captcha-image {
  border: 1px solid #d9d9d9;
  border-radius: 6px;
  padding: 4px;
  background: #fafafa;
  min-width: 80px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.captcha-image:hover {
  border-color: #40a9ff;
  cursor: pointer;
}
</style>
