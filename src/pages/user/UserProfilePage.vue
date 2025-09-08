<template>
  <div class="user-profile-page">
    <a-card title="个人资料" class="profile-card">
      <template #extra>
        <a-button v-if="!isEditing" type="primary" @click="startEdit"> 编辑资料 </a-button>
        <div v-else>
          <a-button type="primary" @click="saveProfile" :loading="saveLoading"> 保存 </a-button>
          <a-button @click="cancelEdit" style="margin-left: 8px"> 取消 </a-button>
        </div>
      </template>

      <a-row :gutter="24">
        <!-- 左侧头像区域 -->
        <a-col :span="8">
          <div class="avatar-section">
            <a-avatar :size="120" :src="formData.userAvatar" class="profile-avatar">
              {{ formData.userName?.[0] || 'U' }}
            </a-avatar>
            <div class="avatar-upload" v-if="isEditing">
              <a-upload
                :show-upload-list="false"
                :custom-request="handleAvatarUpload"
                :before-upload="beforeUpload"
                accept="image/*"
              >
                <a-button type="text" size="small" :loading="avatarLoading">
                  <UploadOutlined v-if="!avatarLoading" />
                  {{ avatarLoading ? '上传中...' : '更换头像' }}
                </a-button>
              </a-upload>
            </div>
          </div>
        </a-col>

        <!-- 右侧表单区域 -->
        <a-col :span="16">
          <a-form
            :model="formData"
            :label-col="{ span: 6 }"
            :wrapper-col="{ span: 18 }"
            class="profile-form"
          >
            <a-form-item label="用户ID">
              <a-input :value="loginUser.id" disabled />
            </a-form-item>

            <a-form-item label="账号">
              <a-input :value="loginUser.userAccount" disabled />
            </a-form-item>

            <a-form-item
              label="用户名"
              :rules="[
                { required: true, message: '请输入用户名' },
                { min: 2, max: 20, message: '用户名长度为2-20个字符' },
              ]"
            >
              <a-input
                v-if="isEditing"
                v-model:value="formData.userName"
                placeholder="请输入用户名"
              />
              <span v-else>{{ loginUser.userName || '未设置' }}</span>
            </a-form-item>

            <a-form-item label="用户简介">
              <a-textarea
                v-if="isEditing"
                v-model:value="formData.userProfile"
                placeholder="请输入个人简介"
                :rows="4"
                :max-length="200"
                show-count
              />
              <div v-else class="profile-text">
                {{ loginUser.userProfile || '暂无简介' }}
              </div>
            </a-form-item>

            <a-form-item label="用户角色">
              <a-tag :color="loginUser.userRole === 'admin' ? 'green' : 'blue'">
                {{ loginUser.userRole === 'admin' ? '管理员' : '普通用户' }}
              </a-tag>
            </a-form-item>

            <a-form-item label="注册时间">
              <span>{{ dayjs(loginUser.createTime).format('YYYY-MM-DD HH:mm:ss') }}</span>
            </a-form-item>

            <a-form-item label="更新时间">
              <span>{{ dayjs(loginUser.updateTime).format('YYYY-MM-DD HH:mm:ss') }}</span>
            </a-form-item>
          </a-form>
        </a-col>
      </a-row>
    </a-card>

    <!-- 修改密码区域 -->
    <a-card title="修改密码" class="password-card" style="margin-top: 24px">
      <a-form
        :model="passwordForm"
        :label-col="{ span: 4 }"
        :wrapper-col="{ span: 12 }"
        @finish="changePassword"
      >
        <a-form-item
          label="原密码"
          name="oldPassword"
          :rules="[{ required: true, message: '请输入原密码' }]"
        >
          <a-input-password v-model:value="passwordForm.oldPassword" placeholder="请输入原密码" />
        </a-form-item>

        <a-form-item
          label="新密码"
          name="newPassword"
          :rules="[
            { required: true, message: '请输入新密码' },
            { min: 6, message: '密码长度至少6位' },
          ]"
        >
          <a-input-password v-model:value="passwordForm.newPassword" placeholder="请输入新密码" />
        </a-form-item>

        <a-form-item
          label="确认密码"
          name="confirmPassword"
          :rules="[{ required: true, message: '请再次输入新密码' }]"
        >
          <a-input-password
            v-model:value="passwordForm.confirmPassword"
            placeholder="请再次输入新密码"
          />
        </a-form-item>

        <a-form-item :wrapper-col="{ offset: 4, span: 12 }">
          <a-button type="primary" html-type="submit" :loading="passwordLoading">
            修改密码
          </a-button>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { message } from 'ant-design-vue'
import { UploadOutlined } from '@ant-design/icons-vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore'
import { updateUserUsingPost } from '@/api/userController'
import { uploadPictureUsingPost } from '@/api/pictureController'
import dayjs from 'dayjs'

const loginUserStore = useLoginUserStore()
const loginUser = loginUserStore.loginUser

// 表单数据
const formData = reactive<API.UserUpdateRequest>({
  id: loginUser.id,
  userName: loginUser.userName,
  userProfile: loginUser.userProfile,
  userAvatar: loginUser.userAvatar,
})

// 密码表单数据
const passwordForm = reactive({
  oldPassword: '',
  newPassword: '',
  confirmPassword: '',
})

// 编辑状态
const isEditing = ref(false)
const saveLoading = ref(false)
const passwordLoading = ref(false)
const avatarLoading = ref(false)

// 原始数据备份
const originalData = ref<API.UserUpdateRequest>({})

// 开始编辑
const startEdit = () => {
  isEditing.value = true
  // 备份原始数据
  originalData.value = { ...formData }
}

// 取消编辑
const cancelEdit = () => {
  isEditing.value = false
  // 恢复原始数据
  Object.assign(formData, originalData.value)
}

// 保存资料
const saveProfile = async () => {
  try {
    saveLoading.value = true

    const res = await updateUserUsingPost(formData)

    if (res.data.code === 0) {
      message.success('保存成功')
      isEditing.value = false
      // 更新登录用户状态
      await loginUserStore.fetchLoginUser()
    } else {
      message.error('保存失败：' + res.data.message)
    }
  } catch (error) {
    message.error('保存失败')
    console.error(error)
  } finally {
    saveLoading.value = false
  }
}

// 修改密码
const changePassword = async () => {
  // 检查密码是否一致
  if (passwordForm.newPassword !== passwordForm.confirmPassword) {
    message.error('两次输入的密码不一致')
    return
  }

  try {
    passwordLoading.value = true

    // 这里需要后端提供修改密码的接口
    // const res = await changePasswordUsingPost({
    //   oldPassword: passwordForm.oldPassword,
    //   newPassword: passwordForm.newPassword
    // })

    // 临时提示，等待后端接口
    message.info('修改密码功能暂未开放，请联系管理员')

    // 重置表单
    Object.assign(passwordForm, {
      oldPassword: '',
      newPassword: '',
      confirmPassword: '',
    })
  } catch (error) {
    message.error('修改密码失败')
    console.error(error)
  } finally {
    passwordLoading.value = false
  }
}

// 头像上传前处理
const beforeUpload = (file: File) => {
  const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png'
  if (!isJpgOrPng) {
    message.error('不支持上传该格式的图片，推荐 jpg 或 png')
    return false
  }

  const isLt2M = file.size / 1024 / 1024 < 2
  if (!isLt2M) {
    message.error('不能上传超过 2M 的图片')
    return false
  }

  return true
}

// 处理头像上传
const handleAvatarUpload = async ({ file }: { file: File }) => {
  avatarLoading.value = true
  try {
    const params: API.uploadPictureUsingPOSTParams = {}
    const res = await uploadPictureUsingPost(params, {}, file)

    // 类型断言，确保 res.data 是 BaseResponsePictureVO_ 类型
    const response = res.data as API.BaseResponsePictureVO_

    if (response.code === 0 && response.data) {
      // 从上传结果中获取图片URL
      const pictureUrl = response.data.url
      if (pictureUrl) {
        formData.userAvatar = pictureUrl
        message.success('头像上传成功')
      } else {
        message.error('获取图片URL失败')
      }
    } else {
      message.error('头像上传失败：' + (response.message || '未知错误'))
    }
  } catch (error) {
    message.error('头像上传失败')
    console.error(error)
  } finally {
    avatarLoading.value = false
  }
}

// 页面加载时获取最新用户信息
onMounted(async () => {
  await loginUserStore.fetchLoginUser()
  // 更新表单数据
  Object.assign(formData, {
    id: loginUser.id,
    userName: loginUser.userName,
    userProfile: loginUser.userProfile,
    userAvatar: loginUser.userAvatar,
  })
})
</script>

<style scoped>
.user-profile-page {
  max-width: 900px;
  margin: 0 auto;
  padding: 24px;
}

.profile-card,
.password-card {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.avatar-section {
  text-align: center;
  padding: 20px;
}

.profile-avatar {
  display: block;
  margin: 0 auto 16px;
  border: 2px solid #f0f0f0;
}

.avatar-upload {
  margin-top: 12px;
}

.profile-form {
  padding-left: 20px;
}

.profile-text {
  min-height: 22px;
  line-height: 1.5;
  color: rgba(0, 0, 0, 0.85);
}

:deep(.ant-form-item-label) {
  font-weight: 500;
}

:deep(.ant-card-head-title) {
  font-size: 18px;
  font-weight: 600;
}
</style>
