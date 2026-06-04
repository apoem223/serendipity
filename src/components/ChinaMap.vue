<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick } from 'vue'
import * as echarts from 'echarts'
import { provinceNameMap, provinceCities } from '@/data/cities.js'
import { provinceCenters } from '@/data/provinceCenters.js'

const props = defineProps({
  footprints: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['province-click'])

const chartRef = ref(null)
let chartInstance = null

// 获取中国地图 GeoJSON
async function fetchChinaMap() {
  try {
    const res = await fetch('/china.json')
    if (!res.ok) throw new Error('Failed to fetch map data')
    return await res.json()
  } catch (e) {
    console.error('获取地图数据失败:', e)
    return null
  }
}

function initChart(geoJson) {
  if (!chartRef.value || !geoJson) return
  echarts.registerMap('china', geoJson)

  chartInstance = echarts.init(chartRef.value, null, { renderer: 'canvas' })

  // 构建标记数据
  const markData = props.footprints.map(fp => {
    const center = provinceCenters[fp.province]
    if (!center) return null
    return {
      name: fp.city,
      value: [...center, fp.images.length],
      province: fp.province,
      city: fp.city,
      images: fp.images
    }
  }).filter(Boolean)

  const option = {
    backgroundColor: 'transparent',
    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(255,255,255,0.95)',
      borderColor: '#eee',
      borderWidth: 1,
      textStyle: { color: '#333' },
      formatter: function(params) {
        if (params.seriesType === 'effectScatter') {
          const data = params.data
          return `<div style="font-weight:600">${data.province} - ${data.city}</div>
                  <div style="margin-top:4px">已上传 ${data.images.length} 张照片</div>`
        }
        const name = params.name
        const hasFootprint = props.footprints.some(fp => fp.province === name || provinceNameMap[name] === fp.province)
        return `<div style="font-weight:600">${name}</div>
                <div style="margin-top:4px">${hasFootprint ? '已有旅行印记 ✈️' : '点击添加旅行印记'}</div>`
      }
    },
    geo: {
      map: 'china',
      roam: true,
      zoom: 1.2,
      scaleLimit: {
        min: 1,
        max: 4
      },
      label: {
        show: true,
        color: '#666',
        fontSize: 10
      },
      itemStyle: {
        areaColor: '#f0f4f8',
        borderColor: '#c5d3e0',
        borderWidth: 1
      },
      emphasis: {
        label: { color: '#333', fontSize: 12, fontWeight: 'bold' },
        itemStyle: {
          areaColor: '#e6eef7',
          borderColor: '#90a4be',
          borderWidth: 1.5,
          shadowBlur: 10,
          shadowColor: 'rgba(0,0,0,0.1)'
        }
      },
      select: {
        itemStyle: { areaColor: '#d4e3f3' }
      }
    },
    series: [
      {
        type: 'effectScatter',
        coordinateSystem: 'geo',
        data: markData,
        symbolSize: function(val) {
          return Math.min(20 + val[2] * 5, 45)
        },
        showEffectOn: 'render',
        rippleEffect: {
          brushType: 'stroke',
          scale: 3,
          period: 4
        },
        label: {
          show: true,
          formatter: '{b}',
          position: 'top',
          color: '#4a5568',
          fontSize: 11,
          fontWeight: 500,
          backgroundColor: 'rgba(255,255,255,0.8)',
          padding: [2, 6],
          borderRadius: 4
        },
        itemStyle: {
          color: '#f5a623',
          shadowBlur: 10,
          shadowColor: 'rgba(245,166,35,0.5)'
        },
        emphasis: {
          scale: true,
          itemStyle: { color: '#ff6b6b' }
        }
      }
    ]
  }

  chartInstance.setOption(option)

  // 点击事件处理
  chartInstance.on('click', function(params) {
    if (params.componentType === 'geo' || params.componentType === 'series') {
      let provinceName = params.name
      if (provinceNameMap[provinceName]) {
        provinceName = provinceNameMap[provinceName]
      }
      const cities = provinceCities[provinceName] || []
      emit('province-click', { province: provinceName, cities })
    }
  })

  window.addEventListener('resize', handleResize)
}

function handleResize() {
  chartInstance && chartInstance.resize()
}

watch(() => props.footprints, () => {
  if (chartInstance) {
    const markData = props.footprints.map(fp => {
      const center = provinceCenters[fp.province]
      if (!center) return null
      return {
        name: fp.city,
        value: [...center, fp.images.length],
        province: fp.province,
        city: fp.city,
        images: fp.images
      }
    }).filter(Boolean)

    chartInstance.setOption({
      series: [{ data: markData }]
    })
  }
}, { deep: true })

onMounted(async () => {
  const geoJson = await fetchChinaMap()
  if (geoJson) {
    await nextTick()
    initChart(geoJson)
  }
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (chartInstance) {
    chartInstance.dispose()
    chartInstance = null
  }
})
</script>

<template>
  <div ref="chartRef" class="china-map">
    <div v-if="!chartInstance" class="map-loading">
      <el-icon size="40" class="loading-icon"><Loading /></el-icon>
      <p>正在加载地图数据...</p>
    </div>
  </div>
</template>

<style scoped>
.china-map {
  width: 100%;
  height: 100%;
  min-height: 600px;
  position: relative;
  touch-action: manipulation;
  -webkit-tap-highlight-color: transparent;
  user-select: none;
}
.map-loading {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #888;
  gap: 12px;
}
.loading-icon {
  animation: spin 1.5s linear infinite;
}
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* 移动端适配 */
@media screen and (max-width: 768px) {
  .china-map {
    min-height: 350px;
    height: calc(100vh - 220px);
  }
}
</style>
