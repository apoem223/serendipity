<script setup>
import { ref, nextTick } from 'vue'
import { ElMessage } from 'element-plus'
import ChinaMap from '@/components/ChinaMap.vue'
import CityModal from '@/components/CityModal.vue'
import TravelCard from '@/components/TravelCard.vue'

const footprints = ref([])

const modalVisible = ref(false)
const currentProvince = ref('')
const currentCities = ref([])

const travelCardRef = ref(null)

function handleProvinceClick({ province, cities }) {
  // 延迟设置弹窗状态，避免在echarts事件处理期间触发Vue更新
  nextTick(() => {
    currentProvince.value = province
    currentCities.value = cities
    modalVisible.value = true
  })
}

function handleAddFootprint({ province, city, images }) {
  const existing = footprints.value.find(fp => fp.province === province && fp.city === city)
  if (existing) {
    existing.images.push(...images)
    ElMessage.success(`已向 ${city} 添加 ${images.length} 张照片`)
  } else {
    footprints.value.push({ province, city, images })
    ElMessage.success(`成功添加 ${province} - ${city} 的旅行印记`)
  }
}

function removeFootprint(index) {
  footprints.value.splice(index, 1)
  ElMessage.info('已删除该旅行印记')
}

function openTravelCard() {
  if (footprints.value.length === 0) {
    ElMessage.warning('请先添加至少一个城市的旅行印记')
    return
  }
  travelCardRef.value?.open()
}
</script>

<template>
  <div class="app-container">
    <!-- 顶部导航 -->
    <header class="app-header">
      <div class="logo">
        <el-icon size="28" color="#f5a623"><Location /></el-icon>
        <h1>serendipity</h1>
      </div>
      <p class="subtitle">记录你走过的每一个角落</p>
    </header>

    <div class="main-layout">
      <!-- 左侧：已添加的足迹列表 -->
      <aside class="sidebar">
        <div class="sidebar-header">
          <h3>我的足迹</h3>
          <el-badge :value="footprints.length" :hidden="footprints.length === 0" />
        </div>
        <div v-if="footprints.length === 0" class="empty-state">
          <el-icon size="32" color="#ccc"><MapLocation /></el-icon>
          <p>点击地图上的省份<br/>开始添加你的旅行印记</p>
        </div>
        <div v-else class="footprint-list">
          <div
            v-for="(fp, index) in footprints"
            :key="`${fp.province}-${fp.city}`"
            class="footprint-card"
          >
            <div class="footprint-info">
              <div class="footprint-location">
                <span class="province">{{ fp.province }}</span>
                <span class="divider">·</span>
                <span class="city">{{ fp.city }}</span>
              </div>
              <div class="footprint-count">{{ fp.images.length }} 张照片</div>
            </div>
            <div class="footprint-thumbs">
              <img
                v-for="(img, idx) in fp.images.slice(0, 3)"
                :key="idx"
                :src="img"
                alt="thumb"
              />
              <div v-if="fp.images.length > 3" class="more-badge">
                +{{ fp.images.length - 3 }}
              </div>
            </div>
            <el-button
              class="delete-btn"
              type="danger"
              link
              size="small"
              @click="removeFootprint(index)"
            >
              <el-icon><Delete /></el-icon>
            </el-button>
          </div>
        </div>
      </aside>

      <!-- 右侧：地图区域 -->
      <main class="map-wrapper">
        <ChinaMap
          :footprints="footprints"
          @province-click="handleProvinceClick"
        />
      </main>
    </div>

    <!-- 右下角保存按钮 -->
    <el-tooltip content="生成旅行卡片" placement="left">
      <el-button
        class="save-btn"
        type="primary"
        size="large"
        circle
        @click="openTravelCard"
      >
        <el-icon size="22"><Download /></el-icon>
      </el-button>
    </el-tooltip>

    <!-- 城市选择弹窗 -->
    <CityModal
      v-model:visible="modalVisible"
      :province="currentProvince"
      :cities="currentCities"
      :existing-footprints="footprints"
      @confirm="handleAddFootprint"
    />

    <!-- 旅行卡片生成器 -->
    <TravelCard ref="travelCardRef" :footprints="footprints" />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: #fafafa;
  color: #1a1a1a;
}
</style>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  padding: 16px 28px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #fff;
  border-bottom: 1px solid #f0f0f0;
  flex-shrink: 0;
}

.logo {
  display: flex;
  align-items: center;
  gap: 10px;
}
.logo h1 {
  font-size: 20px;
  font-weight: 700;
  color: #1a1a1a;
  letter-spacing: -0.5px;
}

.subtitle {
  font-size: 13px;
  color: #888;
  font-weight: 400;
}

.main-layout {
  flex: 1;
  display: flex;
  min-height: 0;
  overflow: hidden;
}

/* 左侧边栏 */
.sidebar {
  width: 300px;
  background: #fff;
  border-right: 1px solid #f0f0f0;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
}
.sidebar-header {
  padding: 16px 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #f5f5f5;
}
.sidebar-header h3 {
  font-size: 15px;
  font-weight: 600;
  color: #333;
}

.empty-state {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 40px 20px;
  text-align: center;
  color: #aaa;
  font-size: 13px;
  line-height: 1.6;
  min-height: 100px;
}

.footprint-list {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.footprint-card {
  background: #fafafa;
  border: 1px solid #f0f0f0;
  border-radius: 12px;
  padding: 12px 14px;
  position: relative;
  transition: all 0.2s;
  overflow: hidden;
}
.footprint-card:hover {
  border-color: #e0e0e0;
  box-shadow: 0 2px 8px rgba(0,0,0,0.04);
}
.footprint-info {
  margin-bottom: 8px;
}
.footprint-location {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  font-weight: 600;
  color: #333;
}
.footprint-location .divider {
  color: #ccc;
  font-weight: 400;
}
.footprint-location .city {
  color: #666;
  font-weight: 500;
}
.footprint-count {
  font-size: 12px;
  color: #999;
  margin-top: 2px;
}
.footprint-thumbs {
  display: flex;
  gap: 6px;
}
.footprint-thumbs img {
  width: 44px;
  height: 44px;
  border-radius: 8px;
  object-fit: cover;
  background: #eee;
}
.more-badge {
  width: 44px;
  height: 44px;
  border-radius: 8px;
  background: #eee;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #666;
  font-weight: 500;
}
.delete-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  opacity: 0;
  transition: opacity 0.2s;
}
.footprint-card:hover .delete-btn {
  opacity: 1;
}

/* 地图区域 */
.map-wrapper {
  flex: 1;
  padding: 10px 16px 16px;
  min-height: 0;
  background: #fafafa;
}

.save-btn {
  position: fixed;
  right: 32px;
  bottom: 32px;
  width: 56px !important;
  height: 56px !important;
  min-width: 56px !important;
  padding: 0 !important;
  border-radius: 50% !important;
  box-shadow: 0 4px 16px rgba(64, 158, 255, 0.35);
  z-index: 100;
  transition: transform 0.2s, box-shadow 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}
.save-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 24px rgba(64, 158, 255, 0.45);
}

/* 响应式设计 - 平板/手机 */
@media screen and (max-width: 768px) {
  .app-header {
    padding: 12px 16px;
  }
  .logo h1 {
    font-size: 18px;
  }
  .subtitle {
    display: none;
  }
  .main-layout {
    flex-direction: column;
  }
  .sidebar {
    width: 100%;
    max-height: 180px;
    border-right: none;
    border-bottom: 1px solid #f0f0f0;
  }
  .empty-state {
    padding: 20px 16px;
    gap: 8px;
  }
  .empty-state p {
    font-size: 12px;
  }
  .footprint-list {
    flex-direction: row;
    overflow-x: auto;
    overflow-y: hidden;
    padding: 8px 12px;
    gap: 8px;
    /* 移动端滚动条样式 */
    scrollbar-width: none; /* Firefox */
    -ms-overflow-style: none; /* IE/Edge */
  }
  .footprint-list::-webkit-scrollbar {
    display: none; /* Chrome/Safari */
  }
  .footprint-card {
    min-width: 160px;
    flex-shrink: 0;
    /* 移动端删除按钮始终显示 */
    padding-right: 36px;
  }
  .delete-btn {
    opacity: 1 !important;
    right: 6px !important;
    top: 50% !important;
    transform: translateY(-50%);
  }
  .map-wrapper {
    flex: 1;
    min-height: calc(100vh - 200px);
    padding: 8px;
    overflow: hidden;
  }
  .save-btn {
    right: 16px !important;
    bottom: 16px !important;
    width: 52px !important;
    height: 52px !important;
    min-width: 52px !important;
  }
}

/* 超小屏幕 */
@media screen and (max-width: 480px) {
  .sidebar {
    max-height: 150px;
  }
  .footprint-card {
    min-width: 140px;
  }
  .footprint-thumbs img {
    width: 36px;
    height: 36px;
  }
}
</style>
