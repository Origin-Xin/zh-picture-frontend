<template>
  <div class="picture-list">
    <a-spin :spinning="loading">
      <MasonryWall
        :items="dataList"
        :column-width="masonryColumnWidth"
        :gap="16"
        :max-columns="6"
        :min-columns="1"
      >
        <template #default="{ item: picture, index }">
          <div class="masonry-item">
            <a-card hoverable @click="doClickPicture(picture)">
              <template #cover>
                <img
                  :alt="picture.name"
                  :src="picture.thumbnailUrl ?? picture.url"
                  loading="lazy"
                  class="masonry-img"
                />
              </template>
              <a-card-meta :title="picture.name">
                <template #description>
                  <a-flex>
                    <a-tag color="green">
                      {{ picture.category ?? '默认' }}
                    </a-tag>
                    <a-tag v-for="tag in picture.tags" :key="tag">
                      {{ tag }}
                    </a-tag>
                  </a-flex>
                </template>
              </a-card-meta>
              <template v-if="showOp" #actions>
                <search-outlined @click="(e) => doSearch(picture, e)" />
                <share-alt-outlined @click="(e) => doShare(picture, e)" />
                <edit-outlined v-if="canEdit" @click="(e) => doEdit(picture, e)" />
                <delete-outlined v-if="canDelete" @click="(e) => doDelete(picture, e)" />
              </template>
            </a-card>
            <ShareModal ref="shareModalRef" :link="shareLink" />
          </div>
        </template>
      </MasonryWall>
    </a-spin>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router'
import { deletePictureUsingPost } from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import ShareModal from '@/pages/ShareModal.vue'
import { ref, computed } from 'vue'
import MasonryWall from '@yeger/vue-masonry-wall'
import {
  DeleteOutlined,
  EditOutlined,
  SearchOutlined,
  ShareAltOutlined,
} from '@ant-design/icons-vue'

// 搜索
const doSearch = (picture, e) => {
  e.stopPropagation()
  window.open(`/search_picture?pictureId=${picture.id}`)
}


// 跳转至图片详情
const router = useRouter()


const doClickPicture = (picture : API.PictureVO) => {
  router.push({
    path: `/picture/${picture.id}`,
  })
}

interface Props {
  dataList?: API.PictureVO[]
  loading?: boolean
  showOp?: boolean
  onReload?: () => void
  canEdit?: boolean
  canDelete?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  dataList: () => [],
  loading: false,
  showOp: false,
  canEdit: false,
  canDelete: false,
})

// 编辑
const doEdit = (picture, e) => {
  e.stopPropagation()
  router.push({
    path: '/add_picture',
    query: {
      id: picture.id,
      spaceId: picture.spaceId,
    },
  })
}

// 删除
const doDelete = async (picture, e) => {
  e.stopPropagation()
  const id = picture.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
    // 让外层刷新
    props?.onReload()
  } else {
    message.error('删除失败')
  }
}

// ----- 分享操作 ----
const shareModalRef = ref()
// 分享链接
const shareLink = ref<string>()
// 分享
const doShare = (picture, e) => {
  // 阻止冒泡
  e.stopPropagation()
  shareLink.value = `${window.location.protocol}//${window.location.host}/picture/${picture.id}`
  if (shareModalRef.value) {
    shareModalRef.value.openModal()
  }
}

// Masonry 响应式列宽：根据容器宽度自适应列数
const masonryColumnWidth = computed(() => 240)
</script>

<style scoped>
.masonry-item {
  margin-bottom: 16px;
}

.masonry-img {
  width: 100%;
  display: block;
  object-fit: cover;
}
</style>
