<script setup>
import { ref, onMounted, computed, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'

const currentYear = ref(2026)
const currentMonth = ref(10)
const pickerYear = ref(2026)
const currentYearMonth = computed(() => `${currentYear.value}年${currentMonth.value}月`)

const isMonthPickerVisible = ref(false)
const monthPickerContainerRef = ref(null)

const toggleMonthPicker = () => {
  if (!isMonthPickerVisible.value) {
    pickerYear.value = currentYear.value
  }
  isMonthPickerVisible.value = !isMonthPickerVisible.value
}

const handleClickOutside = (event) => {
  if (monthPickerContainerRef.value && !monthPickerContainerRef.value.contains(event.target)) {
    isMonthPickerVisible.value = false
  }
}

const prevYear = () => pickerYear.value--
const nextYear = () => pickerYear.value++

const selectMonth = (month) => {
  currentYear.value = pickerYear.value
  currentMonth.value = month
  isMonthPickerVisible.value = false
  
  // 模拟重新计算数据
  showToast(`正在重新计算 ${currentYearMonth.value} 的盘点数据...`, 'info')
  setTimeout(() => {
    // 这里可以加入重新请求或计算图表、列表数据的逻辑
    initCharts()
    showToast('数据已更新', 'success')
  }, 800)
}

// --- 1. 通报文字内容 ---
const reportSummary = computed(() => {
  return `截至目前，辽宁省内各单位及专业已全面启动${currentYearMonth.value}库存盘点工作。全省累计应盘点材料站 324 个，已完成盘点 298 个，总体盘点完成率为 91.98%。其中，沈阳、大连、鞍山等 5 个地市已率先完成 100% 盘点；物资、安监、数字化等 8 个专业已达成全覆盖。整体盘点账实相符率达 99.2%，个别差异已要求相关单位限期整改并提交书面报告。`
})

// --- 2. 图表数据 ---
const cities = ['沈阳', '大连', '鞍山', '抚顺', '本溪', '丹东', '锦州', '营口', '阜新', '辽阳', '盘锦', '铁岭', '朝阳', '葫芦岛']
const cityRates = [100, 100, 100, 100, 100, 98, 96, 95, 92, 90, 88, 85, 82, 80]

const specialties = ['物资', '安监', '数字化', '设备', '调度', '发展', '财务', '人资', '审计', '监察', '党建', '工会', '营销', '建设', '离退休', '保卫', '车辆', '后勤', '档案', '法律', '科技']
const specRates = [100, 100, 100, 100, 100, 100, 100, 100, 98, 97, 95, 95, 94, 92, 90, 88, 86, 85, 82, 80, 75]

const cityChartRef = ref(null)
const specChartRef = ref(null)
let cityChart = null
let specChart = null

const initCharts = () => {
  if (cityChartRef.value) {
    cityChart = echarts.init(cityChartRef.value)
    cityChart.setOption({
      title: { text: '各地市盘点完成率', left: 'center', top: 16, textStyle: { fontSize: 16, fontWeight: 600, color: '#1d1d1f' } },
      tooltip: { trigger: 'axis', formatter: '{b} : {c}%' },
      grid: { left: '2%', right: '2%', bottom: '15%', top: '20%', containLabel: true },
      xAxis: { type: 'category', data: cities, axisLabel: { interval: 0, rotate: 30 } },
      yAxis: { type: 'value', max: 100, axisLabel: { formatter: '{value}%' } },
      series: [{
        type: 'bar',
        data: cityRates,
        barWidth: '40%',
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: '#3b82f6' },
            { offset: 1, color: '#60a5fa' }
          ]),
          borderRadius: [4, 4, 0, 0]
        },
        label: { show: true, position: 'top', formatter: '{c}%', fontSize: 12, color: '#64748b' }
      }]
    })
  }

  if (specChartRef.value) {
    specChart = echarts.init(specChartRef.value)
    specChart.setOption({
      title: { text: '各专业盘点完成率', left: 'center', top: 16, textStyle: { fontSize: 16, fontWeight: 600, color: '#1d1d1f' } },
      tooltip: { trigger: 'axis', formatter: '{b} : {c}%' },
      grid: { left: '2%', right: '2%', bottom: '20%', top: '20%', containLabel: true },
      xAxis: { type: 'category', data: specialties, axisLabel: { interval: 0, rotate: 45 } },
      yAxis: { type: 'value', max: 100, axisLabel: { formatter: '{value}%' } },
      series: [{
        type: 'bar',
        data: specRates,
        barWidth: '40%',
        itemStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: '#10b981' },
            { offset: 1, color: '#34d399' }
          ]),
          borderRadius: [4, 4, 0, 0]
        },
        label: { show: true, position: 'top', formatter: '{c}%', fontSize: 12, color: '#64748b' }
      }]
    })
  }
}

// --- 3. 树形表格数据 ---
// 模拟地市+专业的层级数据
const rawData = [
  {
    id: 'c1', type: 'city', name: '沈阳市', total: 45, completed: 45, expanded: true,
    children: [
      {
        id: 's1-1', type: 'specialty', name: '物资专业', total: 12, completed: 12, expanded: true,
        children: [
          { id: 'i1', type: 'item', name: '沈阳物资一号综合仓', code: 'WH-SY-001', completeDate: '2026-11-10', signDate: '2026-11-11', status: '已完成' },
          { id: 'i2', type: 'item', name: '沈阳浑南物资储备库', code: 'WH-SY-002', completeDate: '2026-11-09', signDate: '2026-11-10', status: '已完成' },
        ]
      },
      {
        id: 's1-2', type: 'specialty', name: '安监专业', total: 8, completed: 8, expanded: false,
        children: [
          { id: 'i3', type: 'item', name: '沈阳安监专项物资库', code: 'WH-SY-003', completeDate: '2026-11-08', signDate: '2026-11-09', status: '已完成' }
        ]
      }
    ]
  },
  {
    id: 'c2', type: 'city', name: '大连市', total: 38, completed: 38, expanded: false,
    children: [
      {
        id: 's2-1', type: 'specialty', name: '物资专业', total: 10, completed: 10, expanded: false,
        children: [
          { id: 'i4', type: 'item', name: '大连金普新区综合仓', code: 'WH-DL-001', completeDate: '2026-11-07', signDate: '2026-11-08', status: '已完成' }
        ]
      }
    ]
  },
  {
    id: 'c3', type: 'city', name: '抚顺市', total: 24, completed: 24, expanded: false,
    children: [
      {
        id: 's3-1', type: 'specialty', name: '数字化专业', total: 5, completed: 5, expanded: false,
        children: [
          { id: 'i5', type: 'item', name: '抚顺数字化设备站', code: 'WH-FS-001', completeDate: '2026-11-06', signDate: '2026-11-07', status: '已完成' }
        ]
      }
    ]
  },
  {
    id: 'c4', type: 'city', name: '葫芦岛市', total: 18, completed: 14, expanded: true,
    children: [
      {
        id: 's4-1', type: 'specialty', name: '建设专业', total: 6, completed: 4, expanded: true,
        children: [
          { id: 'i6', type: 'item', name: '葫芦岛连山基建仓', code: 'WH-HLD-001', completeDate: '2026-11-12', signDate: '-', status: '待签署' },
          { id: 'i7', type: 'item', name: '葫芦岛龙港基建仓', code: 'WH-HLD-002', completeDate: '-', signDate: '-', status: '未盘点' },
        ]
      }
    ]
  }
]

const tableData = ref(rawData)

const flatTableData = computed(() => {
  const result = []
  for (const city of tableData.value) {
    result.push(city)
    if (city.expanded) {
      for (const spec of city.children || []) {
        result.push(spec)
        if (spec.expanded) {
          for (const item of spec.children || []) {
            result.push(item)
          }
        }
      }
    }
  }
  return result
})

const toggleRow = (row) => {
  if (row.type !== 'item') {
    row.expanded = !row.expanded
  }
}

// 展开/收起全部
const expandAll = (expanded) => {
  const traverse = (nodes) => {
    for (const node of nodes) {
      if (node.type !== 'item') {
        node.expanded = expanded
        if (node.children) traverse(node.children)
      }
    }
  }
  traverse(tableData.value)
}

// 弹窗与提示
const toast = ref(null)
let toastTimer = null
const showToast = (message, type = 'info') => {
  toast.value = { message, type }
  clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { toast.value = null }, 2200)
}

const handleAction = (action, row) => {
  if (action === '详情') {
    if (row.status === '未盘点') {
      showToast('盘点任务暂未开始，暂无盘点详情', 'warning')
      return
    }
    showToast(`查看盘点详情: ${row.name}`)
  } else if (action === '报告') {
    if (row.status !== '已完成') {
      showToast('盘点任务未完成，暂无盘点报告', 'warning')
      return
    }
    showToast(`下载盘点报告: ${row.name}`, 'success')
  }
}

const handleResize = () => {
  cityChart?.resize()
  specChart?.resize()
}

onMounted(() => {
  initCharts()
  window.addEventListener('resize', handleResize)
  document.addEventListener('click', handleClickOutside)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  document.removeEventListener('click', handleClickOutside)
  cityChart?.dispose()
  specChart?.dispose()
})
</script>

<template>
  <div class="report-container">
    <!-- 1. 通报描述区域 -->
    <section class="panel summary-panel">
      <div class="summary-content">
        <div class="summary-header">
          <h2 class="summary-title">
            省内材料站
            <div class="month-picker-container" ref="monthPickerContainerRef">
              <span class="clickable-month" @click="toggleMonthPicker">
                {{ currentYearMonth }}
                <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none"><polyline points="6 9 12 15 18 9"></polyline></svg>
              </span>
              <div v-if="isMonthPickerVisible" class="month-dropdown">
                <div class="picker-header">
                  <button class="picker-btn" @click.stop="prevYear">‹</button>
                  <span class="picker-year">{{ pickerYear }}年</span>
                  <button class="picker-btn" @click.stop="nextYear">›</button>
                </div>
                <div class="picker-body">
                  <button
                    v-for="m in 12"
                    :key="m"
                    class="picker-month-btn"
                    :class="{ 'is-active': currentYear === pickerYear && currentMonth === m }"
                    @click.stop="selectMonth(m)"
                  >
                    {{ m }}月
                  </button>
                </div>
              </div>
            </div>
            库存盘点情况通报
          </h2>
          <button class="btn btn--primary btn--small" style="height: 32px;" @click="showToast('导出库存盘点通报成功', 'success')">导出库存盘点通报</button>
        </div>
        <p class="summary-text">{{ reportSummary }}</p>
      </div>
    </section>

    <!-- 2. 图表区域 -->
    <div class="charts-grid">
      <section class="panel chart-panel">
        <div class="chart-scroll-wrapper">
          <div ref="cityChartRef" class="chart-box" style="min-width: 600px;"></div>
        </div>
      </section>
      <section class="panel chart-panel">
        <div class="chart-scroll-wrapper">
          <div ref="specChartRef" class="chart-box" style="min-width: 900px;"></div>
        </div>
      </section>
    </div>

    <!-- 3. 详情列表区域 -->
    <section class="panel list-panel">
      <div class="list-toolbar">
        <h3 class="list-title">盘点明细台账</h3>
        <div class="list-actions">
          <button class="btn btn--ghost btn--sm" @click="expandAll(true)">全部展开</button>
          <button class="btn btn--ghost btn--sm" @click="expandAll(false)">全部收起</button>
          <button class="btn btn--primary btn--sm" @click="showToast('导出报表成功', 'success')">导出明细</button>
        </div>
      </div>
      
      <div class="table-wrap">
        <table class="data-table tree-table">
          <thead>
            <tr>
              <th style="text-align: left; padding-left: 24px;">材料站全称 / 维度</th>
              <th style="width: 140px;">材料站编码</th>
              <th style="width: 140px;">盘点完成日期</th>
              <th style="width: 140px;">审核签署日期</th>
              <th style="width: 110px;">盘点状态</th>
              <th style="width: 160px;" class="cell--actions">操作</th>
            </tr>
          </thead>
          <tbody>
            <tr 
              v-for="row in flatTableData" 
              :key="row.id" 
              :class="[`tree-row-${row.type}`, { 'tree-row-expanded': row.expanded }]"
            >
              <td class="tree-cell">
                <div class="tree-cell-content" :style="{ paddingLeft: row.type === 'city' ? '0px' : row.type === 'specialty' ? '28px' : '56px' }">
                  <button 
                    v-if="row.type !== 'item'" 
                    class="tree-toggle-btn"
                    @click="toggleRow(row)"
                  >
                    <svg v-if="row.expanded" viewBox="0 0 24 24" width="14" height="14"><path fill="currentColor" d="M7 10l5 5 5-5H7z"/></svg>
                    <svg v-else viewBox="0 0 24 24" width="14" height="14"><path fill="currentColor" d="M10 7l5 5-5 5V7z"/></svg>
                  </button>
                  <span v-else class="tree-leaf-icon"></span>
                  <span class="tree-label" :class="{'tree-label-bold': row.type !== 'item'}">{{ row.name }}</span>
                  <span v-if="row.type !== 'item'" class="tree-count">
                    ({{ row.completed }}/{{ row.total }})
                    <span v-if="row.completed === row.total" class="tree-count-badge success">100%</span>
                  </span>
                </div>
              </td>
              <td>{{ row.type === 'item' ? row.code : '-' }}</td>
              <td>{{ row.type === 'item' ? row.completeDate : '-' }}</td>
              <td>{{ row.type === 'item' ? row.signDate : '-' }}</td>
              <td>
                <template v-if="row.type === 'item'">
                  <span class="chip" :class="{
                    'chip--success': row.status === '已完成',
                    'chip--warning': row.status === '待签署',
                    'chip--neutral': row.status === '未盘点'
                  }">{{ row.status }}</span>
                </template>
                <template v-else>-</template>
              </td>
              <td class="cell--actions">
                <template v-if="row.type === 'item'">
                  <div class="actions">
                    <button 
                        class="text-button" 
                        :class="{ 'text-button--disabled': row.status === '未盘点' }"
                        :disabled="row.status === '未盘点'"
                        :title="row.status === '未盘点' ? '盘点任务暂未开始，暂无盘点详情' : ''"
                        @click="handleAction('详情', row)"
                      >盘点详情</button>
                      <button 
                        class="text-button" 
                        :class="{ 'text-button--disabled': row.status !== '已完成' }"
                        :disabled="row.status !== '已完成'"
                        :title="row.status !== '已完成' ? '盘点任务未完成，暂无盘点报告' : ''"
                        @click="handleAction('报告', row)"
                      >盘点报告</button>
                  </div>
                </template>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- Toast 提示 -->
    <transition name="toast-fade">
      <div v-if="toast" class="toast" :class="`toast--${toast.type}`">
        {{ toast.message }}
      </div>
    </transition>
  </div>
</template>

<style scoped>
.report-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 0 0 24px 0;
  height: 100%;
  overflow-y: auto;
}

.panel {
  background: #ffffff;
  border-radius: 12px;
  border: 1px solid var(--line);
  box-shadow: 0 1px 3px rgba(0,0,0,0.02);
}

.summary-panel {
  padding: 24px;
  background: linear-gradient(to right, #ffffff, #f8fafc);
  border-left: 4px solid var(--primary);
  overflow: visible;
}

.summary-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.summary-title {
  font-size: 18px;
  font-weight: 600;
  color: var(--text);
  margin: 0;
  display: flex;
  align-items: center;
  gap: 8px;
}

.clickable-month {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  color: var(--primary);
  background: var(--primary-soft);
  padding: 4px 12px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.clickable-month:hover {
  background: rgba(0, 113, 227, 0.18);
  transform: translateY(-1px);
}

.month-picker-container {
  position: relative;
  display: inline-block;
}

.month-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  left: 50%;
  transform: translateX(-50%);
  background: var(--surface-strong);
  border: 1px solid var(--line);
  border-radius: 12px;
  padding: 16px;
  box-shadow: var(--shadow-md);
  z-index: 100;
  animation: dropdownIn 0.2s ease-out;
  width: 260px;
}

@keyframes dropdownIn {
  from { opacity: 0; transform: translate(-50%, -10px); }
  to { opacity: 1; transform: translate(-50%, 0); }
}

.picker-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.picker-btn {
  width: 28px;
  height: 28px;
  border-radius: 6px;
  background: transparent;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--subtext);
  font-size: 18px;
  transition: background-color 0.2s;
}

.picker-btn:hover {
  background: rgba(15, 23, 42, 0.06);
  color: var(--text);
}

.picker-year {
  font-weight: 600;
  font-size: 15px;
  color: var(--text);
}

.picker-body {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.picker-month-btn {
  padding: 8px 0;
  border-radius: 8px;
  font-size: 13px;
  color: var(--text);
  background: transparent;
  cursor: pointer;
  transition: all 0.2s;
  border: none;
}

.picker-month-btn:hover {
  background: var(--primary-soft);
  color: var(--primary-strong);
}

.picker-month-btn.is-active {
  background: var(--primary);
  color: #fff;
  font-weight: 600;
}

.summary-text {
  font-size: 14px;
  line-height: 1.6;
  color: var(--subtext);
  margin: 0;
  text-indent: 28px;
}

.charts-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.chart-panel {
  height: 380px;
  display: flex;
  flex-direction: column;
}

.chart-scroll-wrapper {
  width: 100%;
  height: 100%;
  overflow-x: auto;
  overflow-y: hidden;
}

.chart-scroll-wrapper::-webkit-scrollbar {
  height: 6px;
}

.chart-scroll-wrapper::-webkit-scrollbar-thumb {
  background: rgba(15, 23, 42, 0.15);
  border-radius: 3px;
}

.chart-scroll-wrapper::-webkit-scrollbar-track {
  background: transparent;
}

.chart-box {
  width: 100%;
  height: 100%;
}

.list-panel {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.list-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 24px;
  border-bottom: 1px solid var(--line);
}

.list-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--text);
  margin: 0;
}

.list-actions {
  display: flex;
  gap: 12px;
}

.tree-table {
  width: 100%;
  min-width: unset;
}

.tree-table th {
  background-color: #f8fafc;
  font-weight: 600;
  color: var(--subtext);
}

.tree-table td {
  padding: 12px 16px;
  vertical-align: middle;
}

.tree-row-city td {
  background-color: #f1f5f9;
  border-bottom: 1px solid #e2e8f0;
}

.tree-row-specialty td {
  background-color: #f8fafc;
  border-bottom: 1px dashed #e2e8f0;
}

.tree-row-item:hover td {
  background-color: #f1f5f9;
}

.tree-cell-content {
  display: flex;
  align-items: center;
  gap: 8px;
}

.tree-toggle-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  border: none;
  background: transparent;
  color: var(--muted);
  cursor: pointer;
  border-radius: 4px;
  transition: all 0.2s;
  padding: 0;
}

.tree-toggle-btn:hover {
  background: rgba(0,0,0,0.05);
  color: var(--text);
}

.tree-leaf-icon {
  width: 20px;
  height: 20px;
  position: relative;
}

.tree-leaf-icon::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 6px;
  height: 6px;
  background-color: var(--muted);
  border-radius: 50%;
  transform: translate(-50%, -50%);
}

.tree-label-bold {
  font-weight: 600;
  color: var(--text);
}

.tree-count {
  font-size: 13px;
  color: var(--subtext);
  margin-left: 4px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.tree-count-badge {
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 10px;
  line-height: 1;
}

.tree-count-badge.success {
  background: #dcfce7;
  color: #166534;
}

.actions {
  display: flex;
  gap: 12px;
  align-items: center;
  justify-content: flex-start;
}

.btn--small {
  padding: 4px 12px;
  font-size: 12px;
  height: 28px;
  line-height: 1;
}
</style>
