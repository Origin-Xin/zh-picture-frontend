<template>
  <!--搜索表单-->
  <a-form layout="inline" :model="searchParams" @finish="doSearch">
    <a-form-item label="账号">
      <a-input v-model:value="searchParams.userAccount" placeholder="输入账号" />
    </a-form-item>
    <a-form-item label="用户名">
      <a-input v-model:value="searchParams.userName" placeholder="输入用户名" />
    </a-form-item>
    <a-form-item>
      <a-button type="primary" html-type="submit">搜索</a-button>
    </a-form-item>
  </a-form>
  <!--表单-->
  <a-table
    :columns="columns"
    :data-source="dataList"
    :pagination="pagination"
    @change="doTableChange"
  >
    <template #headerCell="{ column }">
      <template v-if="column.key === 'name'">
        <span>
          <smile-outlined />
          Name
        </span>
      </template>
    </template>
    <template #bodyCell="{ column, record }">
      <template v-if="column.dataIndex === 'userAvatar'">
        <a-image :src="record.userAvatar" :width="120" />
      </template>
      <template v-else-if="column.dataIndex === 'userName'">
        <div v-if="editingKey === record.id">
          <a-input v-model:value="record.userName" style="margin: -5px 0" />
        </div>
        <div v-else>
          {{ record.userName }}
        </div>
      </template>
      <template v-else-if="column.dataIndex === 'userProfile'">
        <div v-if="editingKey === record.id">
          <a-textarea v-model:value="record.userProfile" :rows="2" style="margin: -5px 0" />
        </div>
        <div v-else>
          {{ record.userProfile }}
        </div>
      </template>
      <template v-else-if="column.dataIndex === 'userRole'">
        <div v-if="editingKey === record.id">
          <a-select v-model:value="record.userRole" style="width: 120px">
            <a-select-option value="user">
              <a-tag color="blue">普通用户</a-tag>
            </a-select-option>
            <a-select-option value="admin">
              <a-tag color="green">管理员</a-tag>
            </a-select-option>
          </a-select>
        </div>
        <div v-else>
          <a-tag v-if="record.userRole === 'admin'" color="green">管理员</a-tag>
          <a-tag v-else color="blue">普通用户</a-tag>
        </div>
      </template>
      <template v-else-if="column.dataIndex === 'createTime'">
        {{ dayjs(record.createTime).format('YYYY-MM-DD HH:mm:ss') }}
      </template>
      <template v-else-if="column.dataIndex === 'updateTime'">
        {{ dayjs(record.createTime).format('YYYY-MM-DD HH:mm:ss') }}
      </template>
      <template v-else-if="column.key === 'action'">
        <div v-if="editingKey === record.id" class="editable-row-operations">
          <a @click="save(record)" style="margin-right: 8px">保存</a>
          <a-popconfirm title="确定取消?" @confirm="cancel">
            <a>取消</a>
          </a-popconfirm>
        </div>
        <div v-else class="editable-row-operations">
          <a @click="edit(record)" style="margin-right: 8px">编辑</a>
          <a-popconfirm title="确定删除?" @confirm="doDelete(record.id)">
            <a style="color: red">删除</a>
          </a-popconfirm>
        </div>
      </template>
    </template>
  </a-table>
</template>
<script lang="ts" setup>
import { SmileOutlined, DownOutlined } from '@ant-design/icons-vue'
import { computed, onMounted, reactive, ref } from 'vue'
import {
  deleteUserUsingPost,
  listUserVoByPageUsingPost,
  updateUserUsingPost,
} from '@/api/userController.ts'
import { message } from 'ant-design-vue'
import dayjs from 'dayjs'
import { cloneDeep } from 'lodash-es'

const columns = [
  {
    title: 'id',
    dataIndex: 'id',
  },
  {
    title: '账号',
    dataIndex: 'userAccount',
  },
  {
    title: '用户名',
    dataIndex: 'userName',
  },
  {
    title: '头像',
    dataIndex: 'userAvatar',
  },
  {
    title: '简介',
    dataIndex: 'userProfile',
  },
  {
    title: '用户角色',
    dataIndex: 'userRole',
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
  },
  {
    title: '更新时间',
    dataIndex: 'updateTime',
  },
  {
    title: '操作',
    key: 'action',
  },
]

// 数据
const dataList = ref<API.UserVO[]>([])
const total = ref(0)

// 编辑相关状态
const editingKey = ref('')
const originalRecord = ref<API.UserVO | null>(null)

// 搜索条件
const searchParams = reactive<API.UserQueryRequest>({
  current: 1,
  pageSize: 10,
})

// 获取数据
const fetchData = async () => {
  const res = await listUserVoByPageUsingPost({
    ...searchParams,
  })
  if (res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
}

// 页面加载时请求一次
onMounted(() => {
  fetchData()
})

// 分页参数
const pagination = computed(() => {
  return {
    current: searchParams.current ?? 1,
    pageSize: searchParams.pageSize ?? 10,
    total: total.value,
    showSizeChanger: true,
    showTotal: (total: number) => `共 ${total} 条`,
  }
})

// 表格变化处理
const doTableChange = (page: any) => {
  searchParams.current = page.current
  searchParams.pageSize = page.pageSize
  fetchData()
}

// 获取数据
const doSearch = () => {
  // 重置页码
  searchParams.current = 1
  fetchData()
}

// 删除数据
const doDelete = async (id: string) => {
  if (!id) {
    return
  }
  const res = await deleteUserUsingPost({ id: parseInt(id) })
  if (res.data.code === 0) {
    message.success('删除成功')
    // 刷新数据
    fetchData()
  } else {
    message.error('删除失败')
  }
}

// 编辑用户
const edit = (record: API.UserVO) => {
  // 保存原始数据，用于取消时恢复
  originalRecord.value = cloneDeep(record)
  editingKey.value = record.id?.toString() || ''
}

// 保存编辑
const save = async (record: API.UserVO) => {
  try {
    const updateData: API.UserUpdateRequest = {
      id: record.id,
      userName: record.userName,
      userProfile: record.userProfile,
      userRole: record.userRole,
      userAvatar: record.userAvatar,
    }

    const res = await updateUserUsingPost(updateData)

    if (res.data.code === 0) {
      message.success('更新成功')
      editingKey.value = ''
      originalRecord.value = null
      // 刷新数据
      fetchData()
    } else {
      message.error('更新失败：' + res.data.message)
    }
  } catch (error) {
    message.error('更新失败')
    console.error(error)
  }
}

// 取消编辑
const cancel = () => {
  // 恢复原始数据
  if (originalRecord.value) {
    const index = dataList.value.findIndex((item: any) => item.id === originalRecord.value?.id)
    if (index > -1) {
      dataList.value[index] = cloneDeep(originalRecord.value) as API.UserVO
    }
  }
  editingKey.value = ''
  originalRecord.value = null
}
</script>

<style scoped>
.editable-row-operations a {
  margin-right: 8px;
}
</style>
