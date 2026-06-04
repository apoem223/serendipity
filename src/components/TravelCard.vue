<script setup>
import { ref, computed } from 'vue'
import { ElMessage } from 'element-plus'
import html2canvas from 'html2canvas'
import MiniMapCard from './MiniMapCard.vue'

const props = defineProps({
  footprints: {
    type: Array,
    default: () => []
  }
})

const visible = ref(false)
const cardTitle = ref('我的旅行印记')
const cardDesc = ref('')
const selectedColor = ref('#ffffff')
const textColor = ref('#1a1a1a')
const fontFamily = ref('system-ui')
const selectedEmoji = ref('📍')
const cardRef = ref(null)
const isGenerating = ref(false)

const emojiOptions = [
  '📍', '📌', '🗺️', '✈️', '🎫', '🧭', '🏔️', '🌊',
  '🚗', '🚂', '⛵', '🗼', '🏯', '🎑', '🌸', '🍜'
]

const colorPresets = [
  '#ffffff', '#fef3c7', '#fde68a', '#fed7aa', '#fecaca',
  '#e9d5ff', '#c7d2fe', '#bfdbfe', '#a5f3fc', '#bbf7d0',
  '#f3f4f6', '#fce7f3', '#ffedd5', '#ecfccb', '#d1fae5'
]

const textColorPresets = [
  '#1a1a1a', '#374151', '#6b7280', '#b91c1c', '#c2410c',
  '#b45309', '#047857', '#0369a6', '#4338ca', '#be185d'
]

const fontOptions = [
  { label: '系统默认', value: 'system-ui' },
  { label: '宋体', value: '"Songti SC", "SimSun", serif' },
  { label: '楷体', value: '"Kaiti SC", "KaiTi", serif' },
  { label: '黑体', value: '"Heiti SC", "SimHei", sans-serif' },
  { label: '优雅衬线', value: '"Georgia", "Times New Roman", serif' },
]

const totalCities = computed(() => props.footprints.length)
const totalImages = computed(() => props.footprints.reduce((sum, fp) => sum + fp.images.length, 0))

// 扁平化所有图片并关联城市信息
const allPhotos = computed(() => {
  const photos = []
  props.footprints.forEach(fp => {
    fp.images.forEach(url => {
      photos.push({
        url,
        city: fp.city,
        province: fp.province
      })
    })
  })
  return photos
})

const cardStyle = computed(() => ({
  backgroundColor: selectedColor.value,
  color: textColor.value,
  fontFamily: fontFamily.value
}))

function open() {
  visible.value = true
}

function close() {
  visible.value = false
}

async function saveCard() {
  if (!cardRef.value) return
  isGenerating.value = true
  try {
    const canvas = await html2canvas(cardRef.value, {
      scale: 2,
      backgroundColor: selectedColor.value,
      useCORS: true,
      allowTaint: true
    })
    const link = document.createElement('a')
    link.download = `旅行印记_${new Date().toLocaleDateString()}.png`
    link.href = canvas.toDataURL('image/png')
    link.click()
    ElMessage.success('卡片已保存')
  } catch (err) {
    console.error(err)
    ElMessage.error('保存失败，请重试')
  } finally {
    isGenerating.value = false
  }
}

defineExpose({ open, close })
</script>

<template>
  <el-dialog
    v-model="visible"
    title="生成旅行卡片"
    width="960px"
    align-center
    class="travel-card-dialog"
    :close-on-click-modal="false"
  >
    <div class="card-editor">
      <!-- 左侧：设置面板 -->
      <div class="settings-panel">
        <div class="setting-group">
          <div class="setting-label">卡片标题</div>
          <el-input v-model="cardTitle" placeholder="输入标题" maxlength="30" show-word-limit />
        </div>

        <div class="setting-group">
          <div class="setting-label">旅行寄语</div>
          <el-input
            v-model="cardDesc"
            type="textarea"
            :rows="3"
            placeholder="写下你想说的话..."
            maxlength="200"
            show-word-limit
          />
        </div>

        <div class="setting-group">
          <div class="setting-label">地图标记 Emoji</div>
          <div class="emoji-grid">
            <div
              v-for="emoji in emojiOptions"
              :key="emoji"
              class="emoji-item"
              :class="{ active: selectedEmoji === emoji }"
              @click="selectedEmoji = emoji"
            >{{ emoji }}</div>
          </div>
        </div>

        <div class="setting-group">
          <div class="setting-label">卡片背景色</div>
          <div class="color-grid">
            <div
              v-for="color in colorPresets"
              :key="color"
              class="color-swatch"
              :class="{ active: selectedColor === color }"
              :style="{ backgroundColor: color }"
              @click="selectedColor = color"
            ></div>
          </div>
          <el-color-picker v-model="selectedColor" style="margin-top: 8px" />
        </div>

        <div class="setting-group">
          <div class="setting-label">文字颜色</div>
          <div class="color-grid">
            <div
              v-for="color in textColorPresets"
              :key="color"
              class="color-swatch"
              :class="{ active: textColor === color }"
              :style="{ backgroundColor: color }"
              @click="textColor = color"
            ></div>
          </div>
          <el-color-picker v-model="textColor" style="margin-top: 8px" />
        </div>

        <div class="setting-group">
          <div class="setting-label">字体风格</div>
          <el-select v-model="fontFamily" style="width: 100%">
            <el-option
              v-for="opt in fontOptions"
              :key="opt.value"
              :label="opt.label"
              :value="opt.value"
            />
          </el-select>
        </div>
      </div>

      <!-- 右侧：卡片预览 -->
      <div class="preview-panel">
        <div ref="cardRef" class="travel-card" :style="cardStyle">
          <!-- 卡片头部 -->
          <div class="card-header">
            <div class="card-avatar">
              <span style="font-size: 24px">{{ selectedEmoji }}</span>
            </div>
            <div class="card-info">
              <h2 class="card-title">{{ cardTitle }}</h2>
              <p class="card-date">{{ new Date().toLocaleDateString('zh-CN') }}</p>
            </div>
          </div>

          <!-- 统计数据 -->
          <div class="card-stats">
            <div class="stat-item">
              <span class="stat-value">{{ totalCities }}</span>
              <span class="stat-label">城市</span>
            </div>
            <div class="stat-item">
              <span class="stat-value">{{ totalImages }}</span>
              <span class="stat-label">足迹</span>
            </div>
          </div>

          <!-- 寄语 -->
          <div v-if="cardDesc" class="card-desc">{{ cardDesc }}</div>

          <!-- 地图区域 -->
          <div class="card-map-area">
            <div class="map-container">
              <MiniMapCard :footprints="footprints" :emoji="selectedEmoji" />
            </div>
          </div>

          <!-- 照片环绕区域 -->
          <div v-if="allPhotos.length > 0" class="card-photos-area">
            <div class="photos-grid">
              <div
                v-for="(photo, idx) in allPhotos.slice(0, 8)"
                :key="idx"
                class="photo-item"
              >
                <div class="photo-img">
                  <img :src="photo.url" :alt="photo.city" crossorigin="anonymous" />
                </div>
                <div class="photo-label">{{ photo.city }}</div>
              </div>
            </div>
            <div v-if="allPhotos.length > 8" class="photos-more">
              +{{ allPhotos.length - 8 }} 张
            </div>
          </div>

          <!-- 页脚 -->
          <div class="card-footer">
            <span class="footer-tag">Travel Footprints</span>
          </div>
        </div>
      </div>
    </div>

    <template #footer>
      <div class="dialog-footer">
        <el-button @click="visible = false">关闭</el-button>
        <el-button type="primary" :loading="isGenerating" @click="saveCard">
          <el-icon><Download /></el-icon>
          保存卡片
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<style scoped>
.travel-card-dialog :deep(.el-dialog) {
  border-radius: 16px;
  max-width: 98vw;
}
.travel-card-dialog :deep(.el-dialog__header) {
  padding: 20px 24px 0;
}
.travel-card-dialog :deep(.el-dialog__body) {
  padding: 20px 24px;
}
.travel-card-dialog :deep(.el-dialog__footer) {
  padding: 12px 24px 20px;
}

.card-editor {
  display: flex;
  gap: 24px;
}

.settings-panel {
  width: 260px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.setting-label {
  font-size: 13px;
  font-weight: 600;
  color: #444;
  margin-bottom: 8px;
}

.emoji-grid {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 6px;
}
.emoji-item {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  border-radius: 8px;
  cursor: pointer;
  border: 2px solid transparent;
  background: #f5f5f5;
  transition: all 0.15s;
}
.emoji-item:hover {
  background: #eee;
}
.emoji-item.active {
  border-color: #409eff;
  background: #e6f0ff;
  transform: scale(1.1);
}

.color-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 6px;
}
.color-swatch {
  width: 28px;
  height: 28px;
  border-radius: 6px;
  cursor: pointer;
  border: 2px solid transparent;
  transition: all 0.15s;
  box-shadow: inset 0 0 0 1px rgba(0,0,0,0.06);
}
.color-swatch.active {
  border-color: #409eff;
  transform: scale(1.15);
}

.preview-panel {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f0f2f5;
  border-radius: 12px;
  padding: 16px;
  min-height: 640px;
}

.travel-card {
  width: 440px;
  min-height: 620px;
  border-radius: 20px;
  padding: 20px;
  box-shadow: 0 10px 40px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  gap: 12px;
  overflow: hidden;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 12px;
}
.card-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: rgba(0,0,0,0.04);
  display: flex;
  align-items: center;
  justify-content: center;
}
.card-title {
  font-size: 18px;
  font-weight: 700;
  margin: 0;
  line-height: 1.3;
}
.card-date {
  font-size: 11px;
  opacity: 0.55;
  margin: 2px 0 0;
}

.card-stats {
  display: flex;
  gap: 20px;
  padding: 10px 0;
  border-top: 1px solid rgba(0,0,0,0.06);
  border-bottom: 1px solid rgba(0,0,0,0.06);
}
.stat-item {
  display: flex;
  align-items: baseline;
  gap: 6px;
}
.stat-value {
  font-size: 22px;
  font-weight: 700;
  line-height: 1;
}
.stat-label {
  font-size: 11px;
  opacity: 0.5;
}

.card-desc {
  font-size: 13px;
  line-height: 1.7;
  opacity: 0.8;
}

.card-map-area {
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid rgba(0,0,0,0.06);
}
.map-container {
  height: 260px;
  width: 100%;
}

.card-photos-area {
  flex: 1;
}
.photos-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}
.photo-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.photo-img {
  width: 100%;
  aspect-ratio: 1;
  border-radius: 8px;
  overflow: hidden;
  background: rgba(0,0,0,0.03);
  border: 1px solid rgba(0,0,0,0.04);
  display: flex;
  align-items: center;
  justify-content: center;
}
.photo-img img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.photo-label {
  font-size: 9px;
  text-align: center;
  color: inherit;
  opacity: 0.6;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.photos-more {
  text-align: center;
  font-size: 11px;
  color: #888;
  margin-top: 6px;
}

.card-footer {
  display: flex;
  justify-content: flex-end;
  padding-top: 4px;
}
.footer-tag {
  font-size: 10px;
  opacity: 0.3;
  letter-spacing: 0.5px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

/* 响应式 */
@media screen and (max-width: 768px) {
  .card-editor {
    flex-direction: column;
  }
  .settings-panel {
    width: 100%;
  }
  .preview-panel {
    min-height: auto;
    padding: 12px;
  }
  .travel-card {
    width: 100%;
    min-height: 500px;
  }
  .photos-grid {
    grid-template-columns: repeat(4, 1fr);
    gap: 6px;
  }
  .photo-img {
    border-radius: 6px;
  }
  .card-map-area {
    border-radius: 10px;
  }
  .map-container {
    height: 200px;
  }
}
</style>
