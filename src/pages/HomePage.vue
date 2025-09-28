<template>
  <div id="homePage">
    <!-- 搜索框 -->
    <div class="search-bar">
      <a-input-search
        placeholder="从海量图片中搜索"
        v-model:value="searchParams.searchText"
        enter-button="搜索"
        size="large"
        @search="doSearch"
      />
    </div>

    <!-- TODO: 分类展示方案（可在未来添加）
         方案一：在热门标签上方添加一个分类菜单，类似于：
         <div class="category-filter">
           <a-radio-group v-model:value="selectedCategory" @change="doSearch">
             <a-radio-button value="all">全部</a-radio-button>
             <a-radio-button v-for="category in categoryList" :key="category" :value="category">
               {{ category }}
             </a-radio-button>
           </a-radio-group>
         </div>

         方案二：在侧边栏中添加分类树形结构
         方案三：使用下拉选择框的形式在搜索框旁边添加分类选择
    -->

    <!-- 热门标签区域 -->
    <div class="popular-tags-section">
      <div class="tags-container">
        <a-spin :spinning="tagsLoading">
          <div class="tags-pentagon">
            <!-- 正五边形布局：5行对称结构 -->
            <div v-for="(row, rowIndex) in pentagonRows" :key="rowIndex" class="pentagon-row">
              <div
                v-for="(tagData, tagIndex) in row"
                :key="tagData.tagName"
                :class="[
                  'tag-item',
                  getTagSizeClass(tagData.count),
                  { 'tag-selected': selectedTags.includes(tagData.tagName) },
                ]"
                :style="{ '--index': rowIndex * 10 + tagIndex }"
                @click="selectTag(tagData.tagName)"
              >
                <span class="tag-hash">#</span>
                <span class="tag-name">{{ tagData.tagName }}</span>
              </div>
            </div>
          </div>
        </a-spin>
      </div>
    </div>
    <!-- 图片列表 -->
    <PictureList :dataList="dataList" :loading="loading" />
    <a-pagination
      style="text-align: right"
      v-model:current="searchParams.current"
      v-model:pageSize="searchParams.pageSize"
      :total="total"
      @change="onPageChange"
    />
  </div>
</template>
<script lang="ts" setup>
// 数据
import { computed, onMounted, reactive, ref } from 'vue'
import {
  listPopularTagCategoryUsingGet,
  listPictureVoByPageUsingPost,
} from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import PictureList from '@/components/PictureList.vue'

const dataList = ref<API.PictureVO[]>([])
const total = ref(0)
const loading = ref(true)

// 热门标签数据
const popularTagsList = ref<Array<{ tagName: string; count: number }>>([])
const tagsLoading = ref(false)
const selectedTags = ref<string[]>([])

/**
 * 计算正五边形布局的行分布
 * 五行结构：10-14-18-14-10 的对称分布（更宽的倒梯形）
 * 为了保持正五边形的美观，标签顺序不是严格按照热度排序
 * 为了保持正五边形的美观，标签顺序不是严格按照热度排序
 */
const pentagonRows = computed(() => {
  const tags = [...popularTagsList.value] // 创建副本以避免修改原数组
  if (tags.length === 0) return []

  // 更宽的倒梯形行分布模式：顶部和底部较窄，中间最宽
  const rowPattern = [10, 14, 18, 14, 10] // 每行的标签数量，总共66个标签

  // 为了美观，重新排列标签顺序
  // 将最热门的标签放在中间行的中央位置，其他标签围绕排列
  const arrangedTags = arrangeTagsForAesthetics(tags)

  const rows: Array<Array<{ tagName: string; count: number }>> = []
  let tagIndex = 0

  for (let rowIndex = 0; rowIndex < rowPattern.length; rowIndex++) {
    const rowTagCount = rowPattern[rowIndex]
    const row: Array<{ tagName: string; count: number }> = []

    for (let i = 0; i < rowTagCount && tagIndex < arrangedTags.length; i++) {
      row.push(arrangedTags[tagIndex])
      tagIndex++
    }

    if (row.length > 0) {
      rows.push(row)
    }
  }

  return rows
})

/**
 * 为了正五边形的美观重新排列标签顺序
 * 将热门标签均匀分布在整个五边形中，而不是集中在某一行
 * 布局结构：[10, 14, 18, 14, 10]，中间行最长，向外递减
 */
function arrangeTagsForAesthetics(
  tags: Array<{ tagName: string; count: number }>,
): Array<{ tagName: string; count: number }> {
  if (tags.length === 0) return []

  // 创建标签副本
  const arranged = new Array(tags.length)

  // 如果标签数量较少，直接返回
  if (tags.length < 5) return [...tags]

  // 按热度排序，获取最热门的标签
  const sortedTags = [...tags].sort((a, b) => b.count - a.count)

  // 定义每行在最终数组中的起始索引和长度
  const rowConfig = [
    { startIndex: 0, length: 10 }, // 第一行：10个标签
    { startIndex: 10, length: 14 }, // 第二行：14个标签
    { startIndex: 24, length: 18 }, // 第三行：18个标签（最长）
    { startIndex: 42, length: 14 }, // 第四行：14个标签
    { startIndex: 56, length: 10 }, // 第五行：10个标签
  ]

  // 创建一个映射来跟踪哪些标签已被放置
  const placedTags = new Set<string>()

  // 重新分布策略：将热门标签均匀分布在每一行中，而不是集中在某一行
  // 每行放置的热门标签数量根据行长度按比例分配

  // 计算每行应该放置的热门标签数量（按行长度比例分配）
  const totalLength = rowConfig.reduce((sum, row) => sum + row.length, 0)
  const hotTagsToDistribute = Math.min(sortedTags.length, Math.floor(tags.length * 0.4)) // 取前40%的热门标签进行分布

  // 按行长度分配热门标签
  const hotTagsPerRow = rowConfig.map((row) =>
    Math.max(1, Math.floor((row.length / totalLength) * hotTagsToDistribute)),
  )

  // 确保总数不超过hotTagsToDistribute
  const totalHotTags = hotTagsPerRow.reduce((sum, count) => sum + count, 0)
  if (totalHotTags > hotTagsToDistribute) {
    // 如果总数超过，调整最大的行
    const maxIndex = hotTagsPerRow.indexOf(Math.max(...hotTagsPerRow))
    hotTagsPerRow[maxIndex] -= totalHotTags - hotTagsToDistribute
  }

  let hotTagIndex = 0

  // 在每行中均匀分布热门标签
  for (let rowIndex = 0; rowIndex < rowConfig.length; rowIndex++) {
    const { startIndex, length } = rowConfig[rowIndex]
    const hotTagCount = hotTagsPerRow[rowIndex]

    // 在当前行中均匀分布热门标签
    for (let i = 0; i < hotTagCount && hotTagIndex < sortedTags.length; i++) {
      const tag = sortedTags[hotTagIndex]
      // 计算在当前行中的位置，尽量均匀分布
      const positionInRow = Math.floor(((i + 0.5) * length) / hotTagCount)
      const actualIndex = startIndex + positionInRow

      // 确保位置有效
      if (actualIndex < arranged.length) {
        arranged[actualIndex] = tag
        placedTags.add(tag.tagName)
        hotTagIndex++
      }
    }
  }

  // 填充剩余位置
  const remainingTags = tags.filter((tag) => !placedTags.has(tag.tagName))
  let remainingIndex = 0

  for (let i = 0; i < arranged.length; i++) {
    // 如果当前位置还没有被填充，则填入剩余标签
    if (!arranged[i] && remainingIndex < remainingTags.length) {
      arranged[i] = remainingTags[remainingIndex]
      remainingIndex++
    }
  }

  // 处理可能的空位（如果有的话）
  for (let i = 0; i < arranged.length; i++) {
    if (!arranged[i] && remainingIndex < tags.length) {
      // 使用原始标签列表填充
      arranged[i] = tags.find((tag) => !placedTags.has(tag.tagName)) || tags[i % tags.length]
    }
  }

  return arranged.filter((tag) => tag !== undefined) as Array<{ tagName: string; count: number }>
}

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({
  current: 1,
  pageSize: 12,
  sortField: 'createTime',
  sortOrder: 'descend',
})

//搜索
const doSearch = () => {
  // 重置搜索条件
  searchParams.current = 1
  fetchData()
}

// 页面加载时请求一次
onMounted(() => {
  fetchData()
  getPopularTags()
})

// 原有的标签和分类列表（保留备用）
// const categoryList = ref<string[]>([])
// const selectedCategory = ref<string>('all')
// const tagList = ref<string[]>([])
// const selectedTagList = ref<boolean[]>([])

/**
 * 获取热门标签
 */
const getPopularTags = async () => {
  tagsLoading.value = true
  try {
    const res = await listPopularTagCategoryUsingGet()
    if (res.data.code === 0 && res.data.data) {
      const tags = res.data.data.tagList ?? []
      // 后端已排序，按返回顺序设置热度等级（index越小越热门）
      popularTagsList.value = tags.map((tag: string, index: number) => ({
        tagName: tag,
        // 按后端排序顺序设置热度：第1个最热(1000)，第2个次之(900)，以此类推
        count: Math.max(1000 - index * 50, 100),
      }))
    } else {
      message.error('获取热门标签失败，' + res.data.message)
    }
  } catch {
    message.error('获取热门标签失败')
  }
  tagsLoading.value = false
}

/**
 * 根据标签热门程度返回样式类名
 */
const getTagSizeClass = (count: number): string => {
  if (count >= 800) return 'tag-extra-large'
  if (count >= 600) return 'tag-large'
  if (count >= 400) return 'tag-medium'
  if (count >= 200) return 'tag-small'
  return 'tag-extra-small'
}

/**
 * 选择标签
 */
const selectTag = (tagName: string) => {
  const index = selectedTags.value.indexOf(tagName)
  if (index > -1) {
    // 取消选择
    selectedTags.value.splice(index, 1)
  } else {
    // 添加选择
    selectedTags.value.push(tagName)
  }
  doSearch()
}

const fetchData = async () => {
  loading.value = true
  // 转换搜索参数
  const params: API.PictureQueryRequest = {
    ...searchParams,
    tags: [...selectedTags.value], // 使用新的标签选择方式
  }

  const res = await listPictureVoByPageUsingPost(params)
  if (res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
  loading.value = false
}

// 分页参数
const onPageChange = (page: number, pageSize: number) => {
  searchParams.current = page
  searchParams.pageSize = pageSize
  fetchData()
}
</script>

<style scoped>
#homePage {
  margin-bottom: 16px;
}

#homePage .search-bar {
  max-width: 480px;
  margin: 0 auto 24px;
}

/* 热门标签区域样式 */
.popular-tags-section {
  margin-bottom: 32px;
  padding: 32px 24px;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(8px);
  border-radius: 16px;
  margin: 0 20px 24px 20px;
}

.tags-container {
  display: flex;
  justify-content: center;
  width: 100%;
}

/* 正五边形布局 */
.tags-pentagon {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  max-width: 1000px;
  width: 100%;
}

.pentagon-row {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  width: 100%;
}

/* 每行的对称间距 */
.pentagon-row:nth-child(1) {
  /* 第一行：10个标签 */
  gap: 16px;
}

.pentagon-row:nth-child(2) {
  /* 第二行：14个标签 */
  gap: 14px;
}

.pentagon-row:nth-child(3) {
  /* 第三行：18个标签（中间最宽） */
  gap: 12px;
}

.pentagon-row:nth-child(4) {
  /* 第四行：14个标签 */
  gap: 14px;
}

.pentagon-row:nth-child(5) {
  /* 第五行：10个标签 */
  gap: 16px;
}

/* 标签项基础样式 */
.tag-item {
  display: inline-flex;
  align-items: center;
  padding: 6px 12px;
  background: transparent;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  user-select: none;
  color: #1890ff; /* 基础蓝色 */
  writing-mode: horizontal-tb; /* 确保文字横向展示 */
  text-orientation: mixed; /* 文字方向 */
  white-space: nowrap; /* 防止文字换行 */
  animation: float 6s ease-in-out infinite;
  animation-delay: calc(var(--index, 0) * 0.2s);
}

.tag-item:hover {
  background: rgba(24, 144, 255, 0.1);
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 4px 12px rgba(24, 144, 255, 0.15);
  animation-play-state: paused;
}

.tag-item.tag-selected {
  background: rgba(24, 144, 255, 0.8);
  color: white;
  animation-play-state: paused;
}

.tag-hash {
  font-weight: bold;
  margin-right: 4px;
  opacity: 0.8;
}

.tag-name {
  font-weight: 500;
}

/* 根据热门程度的不同大小和颜色深度 */
.tag-extra-large {
  font-size: 24px;
  padding: 14px 20px;
  font-weight: 700;
  color: #0050b3; /* 最深蓝色 - 最热门 */
}

.tag-large {
  font-size: 20px;
  padding: 12px 18px;
  font-weight: 650;
  color: #096dd9; /* 深蓝色 */
}

.tag-medium {
  font-size: 16px;
  padding: 8px 14px;
  font-weight: 600;
  color: #1890ff; /* 标准蓝色 */
}

.tag-small {
  font-size: 13px;
  padding: 6px 12px;
  font-weight: 500;
  color: #40a9ff; /* 浅蓝色 */
}

.tag-extra-small {
  font-size: 11px;
  padding: 4px 8px;
  font-weight: 400;
  color: #91d5ff; /* 最浅蓝色 - 最少热门 */
}

/* 响应式设计 */
@media (max-width: 768px) {
  .popular-tags-section {
    margin: 0 8px 20px 8px;
    padding: 20px 12px;
  }

  .tags-pentagon {
    max-width: 100%;
    gap: 12px;
  }

  .pentagon-row {
    gap: 8px;
  }

  .pentagon-row:nth-child(1),
  .pentagon-row:nth-child(5) {
    gap: 10px;
  }

  .pentagon-row:nth-child(2),
  .pentagon-row:nth-child(4) {
    gap: 9px;
  }

  .pentagon-row:nth-child(3) {
    gap: 8px;
  }

  .tag-extra-large {
    font-size: 22px;
    padding: 12px 16px;
  }

  .tag-large {
    font-size: 18px;
    padding: 10px 14px;
  }

  .tag-medium {
    font-size: 14px;
    padding: 6px 10px;
  }

  .tag-small {
    font-size: 12px;
    padding: 5px 8px;
  }

  .tag-extra-small {
    font-size: 10px;
    padding: 3px 6px;
  }
}

#homePage .ant-pagination {
  margin-top: 24px;
  text-align: center;
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0) translateX(0);
  }
  25% {
    transform: translateY(-3px) translateX(1px);
  }
  50% {
    transform: translateY(0) translateX(-1px);
  }
  75% {
    transform: translateY(-2px) translateX(1px);
  }
}
</style>
