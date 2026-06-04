<script setup>
import { ref, computed } from 'vue'
import { ElMessage } from 'element-plus'

const props = defineProps({
  visible: Boolean,
  province: {
    type: String,
    default: ''
  },
  cities: {
    type: Array,
    default: () => []
  },
  existingFootprints: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['update:visible', 'confirm'])

const selectedCity = ref('')
const fileList = ref([])

const existingForProvince = computed(() => {
  return props.existingFootprints.filter(fp => fp.province === props.province)
})

const dialogVisible = computed({
  get: () => props.visible,
  set: (val) => emit('update:visible', val)
})

function handleClose() {
  selectedCity.value = ''
  fileList.value = []
}

function handleFileChange(uploadFile) {
  const reader = new FileReader()
  reader.onload = (e) => {
    fileList.value.push({
      name: uploadFile.name,
      url: e.target.result,
      raw: uploadFile.raw
    })
  }
  reader.readAsDataURL(uploadFile.raw)
}

function removeFile(index) {
  fileList.value.splice(index, 1)
}

function handleConfirm() {
  if (!selectedCity.value) {
    ElMessage.warning('请选择城市')
    return
  }
  if (fileList.value.length === 0) {
    ElMessage.warning('请至少上传一张图片')
    return
  }
  emit('confirm', {
    province: props.province,
    city: selectedCity.value,
    images: fileList.value.map(f => f.url)
  })
  dialogVisible.value = false
  selectedCity.value = ''
  fileList.value = []
}
</script>

<template>
  <el-dialog
    v-model="dialogVisible"
    :title="`${province} - 选择城市`"
    width="560px"
    align-center
    @closed="handleClose"
    class="city-modal"
  >
    <div class="modal-body">
      <!-- 已有印记提示 -->
      <div v-if="existingForProvince.length > 0" class="existing-section">
        <div class="section-title">已添加的城市</div>
        <div class="existing-tags">
          <el-tag
            v-for="fp in existingForProvince"
            :key="fp.city"
            type="success"
            effect="light"
            class="existing-tag"
          >
            {{ fp.city }} ({{ fp.images.length }}张)
          </el-tag>
        </div>
      </div>

      <!-- 城市选择 -->
      <div class="section">
        <div class="section-title">选择城市</div>
        <el-select
          v-model="selectedCity"
          placeholder="请选择一个城市"
          clearable
          filterable
          style="width: 100%"
          size="large"
        >
          <el-option
            v-for="city in cities"
            :key="city"
            :label="city"
            :value="city"
          />
        </el-select>
      </div>

      <!-- 图片上传 -->
      <div class="section">
        <div class="section-title">上传旅行照片</div>
        <el-upload
          drag
          action="#"
          :auto-upload="false"
          :on-change="handleFileChange"
          :show-file-list="false"
          multiple
          accept="image/*"
          class="upload-area"
        >
          <el-icon class="upload-icon"><Upload /></el-icon>
          <div class="upload-text">拖拽图片到此处，或 <em>点击上传</em></div>
          <div class="upload-tip">支持 jpg、png、gif 格式</div>
        </el-upload>

        <!-- 已上传图片预览 -->
        <div v-if="fileList.length > 0" class="preview-list">
          <div v-for="(file, index) in fileList" :key="index" class="preview-item">
            <img :src="file.url" alt="preview" />
            <div class="preview-overlay" @click="removeFile(index)">
              <el-icon><Delete /></el-icon>
            </div>
          </div>
        </div>
      </div>
    </div>

    <template #footer>
      <div class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleConfirm" :disabled="!selectedCity || fileList.length === 0">
          确认添加
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<style scoped>
.city-modal :deep(.el-dialog) {
  border-radius: 16px;
  overflow: hidden;
  max-width: 95vw;
}
.city-modal :deep(.el-dialog__header) {
  padding: 20px 24px 0;
  margin-right: 0;
}
.city-modal :deep(.el-dialog__title) {
  font-weight: 600;
  font-size: 18px;
  color: #1a1a1a;
}
.city-modal :deep(.el-dialog__body) {
  padding: 20px 24px;
}
.city-modal :deep(.el-dialog__footer) {
  padding: 12px 24px 20px;
}

.modal-body {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.section-title {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin-bottom: 10px;
}

.existing-section {
  background: #f6ffed;
  border: 1px solid #b7eb8f;
  border-radius: 10px;
  padding: 14px 16px;
}
.existing-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.existing-tag {
  border-radius: 20px;
}

.upload-area :deep(.el-upload-dragger) {
  width: 100%;
  border-radius: 12px;
  border: 2px dashed #d0d7de;
  background: #fafbfc;
  transition: all 0.2s;
}
.upload-area :deep(.el-upload-dragger:hover) {
  border-color: #409eff;
  background: #f0f7ff;
}
.upload-icon {
  font-size: 36px;
  color: #8c959f;
  margin-bottom: 8px;
}
.upload-text {
  color: #555;
  font-size: 14px;
}
.upload-text em {
  color: #409eff;
  font-style: normal;
  font-weight: 500;
}
.upload-tip {
  font-size: 12px;
  color: #8c959f;
  margin-top: 6px;
}

.preview-list {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin-top: 14px;
}
.preview-item {
  position: relative;
  aspect-ratio: 1;
  border-radius: 10px;
  overflow: hidden;
  cursor: pointer;
}
.preview-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.preview-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.2s;
}
.preview-item:hover .preview-overlay {
  opacity: 1;
}
.preview-overlay .el-icon {
  color: #fff;
  font-size: 20px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

/* 移动端适配 */
@media screen and (max-width: 768px) {
  .city-modal :deep(.el-dialog) {
    width: 95vw !important;
    max-width: 95vw;
    margin: 10px auto;
  }
  .city-modal :deep(.el-dialog__body) {
    padding: 16px;
  }
  .preview-list {
    grid-template-columns: repeat(3, 1fr);
  }
}
</style>
