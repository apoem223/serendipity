<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick } from 'vue'
import * as echarts from 'echarts'
import { provinceCenters } from '@/data/provinceCenters.js'

const props = defineProps({
  footprints: {
    type: Array,
    default: () => []
  },
  emoji: {
    type: String,
    default: '📍'
  }
})

const mapRef = ref(null)
let chartInstance = null
let geoJson = null

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

function buildOption() {
  const markData = props.footprints
    .map(fp => {
      const center = provinceCenters[fp.province]
      if (!center || !fp.images || fp.images.length === 0) return null
      return {
        name: fp.city,
        value: center,
        province: fp.province,
        city: fp.city,
        images: fp.images
      }
    })
    .filter(Boolean)

  return {
    backgroundColor: 'transparent',
    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(255,255,255,0.95)',
      borderColor: '#eee',
      borderWidth: 1,
      borderRadius: 8,
      padding: [8, 12],
      textStyle: { color: '#333', fontSize: 12 },
      formatter: function(params) {
        const d = params.data
        if (!d || !d.images) return ''
        return `<b>${d.province} - ${d.city}</b><br/><span style="color:#888">${d.images.length} 张照片</span>`
      }
    },
    geo: {
      map: 'china',
      roam: false,
      zoom: 1.0,
      silent: true,
      label: { show: false },
      itemStyle: {
        areaColor: '#e8f4fc',
        borderColor: '#b8d4e8',
        borderWidth: 0.8
      },
      emphasis: {
        itemStyle: {
          areaColor: '#d4eaf7',
          borderColor: '#90c5e0'
        }
      }
    },
    series: [
      {
        type: 'scatter',
        coordinateSystem: 'geo',
        data: markData,
        symbol: `image://data:image/svg+xml;charset=UTF-8,${encodeURIComponent('<svg xmlns="http://www.w3.org/2000/svg" width="30" height="30" viewBox="0 0 30 30"><text x="50%" y="50%" dominant-baseline="central" text-anchor="middle" font-size="22">' + props.emoji + '</text></svg>')}`,
        symbolSize: 28,
        symbolOffset: [0, 0],
        z: 10
      }
    ]
  }
}

function initChart() {
  if (!mapRef.value || !geoJson) return
  echarts.registerMap('china', geoJson)
  chartInstance = echarts.init(mapRef.value, null, { renderer: 'canvas' })
  chartInstance.setOption(buildOption())
}

watch(() => props.footprints, () => {
  if (chartInstance && geoJson) {
    chartInstance.setOption(buildOption(), { notMerge: true })
  }
}, { deep: true })

watch(() => props.emoji, () => {
  if (chartInstance && geoJson) {
    chartInstance.setOption(buildOption(), { notMerge: true })
  }
})

onMounted(async () => {
  geoJson = await fetchChinaMap()
  if (geoJson) {
    await nextTick()
    initChart()
  }
})

onUnmounted(() => {
  if (chartInstance) {
    chartInstance.dispose()
    chartInstance = null
  }
})
</script>

<template>
  <div ref="mapRef" class="mini-map"></div>
</template>

<style scoped>
.mini-map {
  width: 100%;
  height: 100%;
  min-height: 200px;
  border-radius: 12px;
  overflow: hidden;
}
</style>
