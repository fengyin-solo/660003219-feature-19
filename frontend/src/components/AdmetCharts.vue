<template>
  <div v-if="store.radarChartData && store.riskComparisonData" class="space-y-4">
    <div class="bg-slate-900 rounded-lg p-3">
      <h4 class="text-xs font-bold text-slate-400 mb-2">多维度性质雷达图</h4>
      <v-chart class="h-64" :option="radarOption" autoresize />
    </div>
    <div class="bg-slate-900 rounded-lg p-3">
      <h4 class="text-xs font-bold text-slate-400 mb-2">分子风险对比</h4>
      <v-chart class="h-56" :option="barOption" autoresize />
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { RadarChart, BarChart } from 'echarts/charts'
import {
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent
} from 'echarts/components'
import VChart from 'vue-echarts'
import { useMoleculeStore } from '../store/molecule'

use([
  CanvasRenderer,
  RadarChart,
  BarChart,
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent
])

const store = useMoleculeStore()

const radarColors = ['#22c55e', '#3b82f6', '#f97316']

const radarOption = computed(() => {
  const data = store.radarChartData
  if (!data) return {}

  const series = [
    {
      name: data.current.name,
      value: data.current.data,
      symbol: 'circle',
      symbolSize: 6,
      lineStyle: { width: 2, color: radarColors[0] },
      itemStyle: { color: radarColors[0] },
      areaStyle: {
        color: radarColors[0],
        opacity: 0.25
      }
    },
    ...data.similar.map((sim, idx) => ({
      name: sim.name,
      value: sim.data,
      symbol: 'circle',
      symbolSize: 5,
      lineStyle: { width: 1.5, color: radarColors[idx + 1], type: 'dashed' as const },
      itemStyle: { color: radarColors[idx + 1] },
      areaStyle: {
        color: radarColors[idx + 1],
        opacity: 0.12
      }
    }))
  ]

  return {
    tooltip: {
      trigger: 'item',
      backgroundColor: 'rgba(30, 41, 59, 0.95)',
      borderColor: '#475569',
      textStyle: { color: '#e2e8f0', fontSize: 11 }
    },
    legend: {
      data: [data.current.name, ...data.similar.map(s => s.name)],
      textStyle: { color: '#94a3b8', fontSize: 10 },
      bottom: 0,
      itemWidth: 12,
      itemHeight: 8
    },
    radar: {
      indicator: data.indicators,
      shape: 'polygon',
      splitNumber: 4,
      center: ['50%', '45%'],
      radius: '65%',
      axisName: {
        color: '#94a3b8',
        fontSize: 10
      },
      splitLine: {
        lineStyle: { color: '#334155' }
      },
      splitArea: {
        show: true,
        areaStyle: {
          color: ['rgba(30, 41, 59, 0.3)', 'rgba(51, 65, 85, 0.2)']
        }
      },
      axisLine: {
        lineStyle: { color: '#475569' }
      }
    },
    series: [{
      type: 'radar',
      data: series
    }]
  }
})

const barOption = computed(() => {
  const data = store.riskComparisonData
  if (!data) return {}

  const riskColors = data.riskScores.map(s =>
    s > 75 ? '#ef4444' : s > 50 ? '#f97316' : s > 30 ? '#eab308' : '#22c55e'
  )

  return {
    tooltip: {
      trigger: 'axis',
      backgroundColor: 'rgba(30, 41, 59, 0.95)',
      borderColor: '#475569',
      textStyle: { color: '#e2e8f0', fontSize: 11 },
      axisPointer: { type: 'shadow' }
    },
    legend: {
      data: ['综合风险', '生物利用度', '脂溶性'],
      textStyle: { color: '#94a3b8', fontSize: 10 },
      top: 0,
      itemWidth: 12,
      itemHeight: 8
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '12%',
      top: '18%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: data.categories,
      axisLine: { lineStyle: { color: '#475569' } },
      axisLabel: {
        color: '#94a3b8',
        fontSize: 10,
        interval: 0,
        rotate: 25
      }
    },
    yAxis: [
      {
        type: 'value',
        name: '风险/利用度(%)',
        nameTextStyle: { color: '#64748b', fontSize: 9 },
        max: 100,
        axisLine: { lineStyle: { color: '#475569' } },
        axisLabel: { color: '#94a3b8', fontSize: 10 },
        splitLine: { lineStyle: { color: '#334155', type: 'dashed' } }
      },
      {
        type: 'value',
        name: 'LogP',
        nameTextStyle: { color: '#64748b', fontSize: 9 },
        max: 6,
        axisLine: { lineStyle: { color: '#475569' } },
        axisLabel: { color: '#94a3b8', fontSize: 10 },
        splitLine: { show: false }
      }
    ],
    series: [
      {
        name: '综合风险',
        type: 'bar',
        data: data.riskScores.map((value, idx) => ({
          value,
          itemStyle: {
            color: riskColors[idx],
            borderRadius: [3, 3, 0, 0]
          }
        })),
        barWidth: '22%',
        label: {
          show: true,
          position: 'top',
          color: '#e2e8f0',
          fontSize: 9,
          formatter: '{c}'
        }
      },
      {
        name: '生物利用度',
        type: 'bar',
        data: data.bioavailabilities,
        barWidth: '22%',
        itemStyle: {
          color: '#3b82f6',
          borderRadius: [3, 3, 0, 0]
        },
        label: {
          show: true,
          position: 'top',
          color: '#e2e8f0',
          fontSize: 9,
          formatter: '{c}%'
        }
      },
      {
        name: '脂溶性',
        type: 'bar',
        yAxisIndex: 1,
        data: data.logPs,
        barWidth: '22%',
        itemStyle: {
          color: '#8b5cf6',
          borderRadius: [3, 3, 0, 0]
        },
        label: {
          show: true,
          position: 'top',
          color: '#e2e8f0',
          fontSize: 9,
          formatter: '{c}'
        }
      }
    ]
  }
})
</script>
