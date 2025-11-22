<template>
  <div class="chart-card">
    <BaseChart
      :options="chartOption"
      theme="light"
      :loading="loading || internalLoading"
      @click="handleChartClick"
    />
  </div>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'
import * as echarts from 'echarts'

const store = useDashboardStore()

const { loading } = defineProps(['loading'])

const internalLoading = ref(false)

/**
 * 处理图表点击：下钻
 */
const handleChartClick = async (params) => {
  // 你的 store 里已经做了 ">=3" 的判断，
  // 但在 UI 层判断一下可以避免不必要的函数调用
  if (store.currentLevel < 3) {
    internalLoading.value = true
    try {
      await store.drillDown(params.name)
    } finally {
      internalLoading.value = false
    }
  } else {
    console.log('已经是最后一级，无法下钻')
  }
}

const chartOption = computed(() => {
  const data = store.middleChartData || []

  // 提取 Y轴(名称) 和 Series(数值)
  const categoryData = data.map((item) => item.name)
  const seriesData = data.map((item) => item.value)

  return {
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' },
      formatter: '{b}: {c}',
    },
    grid: {
      top: '5%',
      left: '2%',
      right: '15%', // 右侧留宽一点给数值标签
      bottom: '0%',
      containLabel: true,
    },
    xAxis: {
      type: 'value',
      show: true,
      splitLine: { show: true },
      // max: function (value) {
      //   return value.max * 1.1
      // },
    },
    yAxis: [
      {
        type: 'category',
        data: categoryData,
        inverse: false, // 关键：列表通常从上到下，inverse: true 让第一个数据在顶部
        axisLine: { show: true },
        axisTick: { show: false },
        axisLabel: {
          color: '#146fd7',
          fontSize: 13,
          width: 110,
          overflow: 'truncate', // 文字过长省略
          background: '#d3efff',
        },
      },
      {
        type: 'category',
        inverse: true,
        axisTick: 'none',
        axisLine: 'none',
        show: true,
        axisLabel: {
          textStyle: {
            color: '#146fd7',
            fontSize: 14,
          },
          formatter: function (value) {
            return value
          },
        },
        data: seriesData,
      },
    ],
    series: [
      {
        name: '数量',
        type: 'bar',
        data: seriesData,
        barWidth: 5, // 柱子宽度
        showBackground: true, // 开启背景轨道
        backgroundStyle: {
          color: '#d2e8ff',
          borderRadius: 7,
        },
        itemStyle: {
          borderRadius: 7,
          // 蓝色渐变
          color: new echarts.graphic.LinearGradient(0, 0, 1, 0, [
            { offset: 0, color: '#6eafff' },
            { offset: 1, color: '#0072ff ' },
          ]),
        },
      },
    ],
  }
})

// --- 3. 生命周期 ---
</script>

<style scoped>
.chart-card {
  width: 100%;
  height: 100%;
  background: #fff;
  border-radius: 12px;
  padding: 16px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}
</style>
