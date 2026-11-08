<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const activeTab = ref('stock') // 默认显示存量物资

const createEmptyForm = () => ({
  unit1: '',
  unit2: '',
  unit3: '',
  stationName: '',
  stationCode: '',
  inboundTime: '',
  projectName: '',
  poNumber: '',
  materialCode: '',
  materialDesc: ''
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())

const tableColumns = ref([
  { key: 'selection', label: '', width: '40px', sortable: false, visible: true },
  { key: 'index', label: '序号', width: '60px', sortable: false, visible: true },
  { key: 'unit1', label: '一级单位', width: '150px', sortable: true, visible: true },
  { key: 'unit2', label: '二级单位/部门', width: '150px', sortable: true, visible: true },
  { key: 'unit3', label: '三级部门', width: '120px', sortable: true, visible: true },
  { key: 'stationName', label: '材料站全称', width: '180px', sortable: true, visible: true },
  { key: 'stationCode', label: '材料站编码', width: '120px', sortable: true, visible: true },
  { key: 'materialMajor', label: '物料大类', width: '120px', sortable: true, visible: true },
  { key: 'materialMedium', label: '物料中类', width: '120px', sortable: true, visible: true },
  { key: 'materialMinor', label: '物料小类', width: '120px', sortable: true, visible: true },
  { key: 'materialCode', label: '物料编码', width: '120px', sortable: true, visible: true },
  { key: 'materialDesc', label: '物料描述', width: '180px', sortable: true, visible: true },
  { key: 'initialQty', label: '初始数量', width: '100px', sortable: true, visible: true },
  { key: 'remainingQty', label: '剩余数量', width: '100px', sortable: true, visible: true },
  { key: 'unit', label: '计量单位', width: '80px', sortable: true, visible: true },
  { key: 'unitPrice', label: '单价/元', width: '100px', sortable: true, visible: true },
  { key: 'totalPrice', label: '总价/元', width: '100px', sortable: true, visible: true },
  { key: 'inboundTime', label: '入站时间', width: '150px', sortable: true, visible: true },
  { key: 'projectName', label: '项目名称', width: '160px', sortable: true, visible: true },
  { key: 'poNumber', label: '采购订单编号', width: '140px', sortable: true, visible: true },
  { key: 'wbsCode', label: 'WBS编码', width: '120px', sortable: true, visible: true },
  { key: 'actions', label: '操作', width: '100px', sortable: false, visible: true },
])

const generateMockData = () => {
  return Array.from({ length: 30 }, (_, i) => {
    return {
      id: i + 1,
      unit1: '国网辽宁省电力有限公司',
      unit2: i % 2 === 0 ? '国网沈阳供电公司' : '国网大连供电公司',
      unit3: '物资部',
      stationName: i % 2 === 0 ? '沈阳和平区综合材料站' : '大连中山区综合材料站',
      stationCode: `STA${String((i % 2) + 1).padStart(3, '0')}`,
      materialMajor: '一次设备',
      materialMedium: '变压器',
      materialMinor: '配电变压器',
      materialCode: `MAT${String(i + 1).padStart(5, '0')}`,
      materialDesc: `10kV油浸式变压器型号${i}`,
      initialQty: (i + 1) * 10,
      remainingQty: (i + 1) * 5,
      unit: '台',
      unitPrice: 15000.00,
      totalPrice: (i + 1) * 10 * 15000.00,
      inboundTime: `2026-11-${String((i % 28) + 1).padStart(2, '0')} ${String((i % 14) + 8).padStart(2, '0')}:30`,
      projectName: '配网改造工程' + (i % 3 === 0 ? '一期' : '二期'),
      poNumber: `PO202611${String(i + 1).padStart(4, '0')}`,
      wbsCode: `WBS-2026-${String(i + 1).padStart(3, '0')}`,
      type: i % 3 === 0 ? 'incremental' : 'stock'
    }
  })
}

const mockData = generateMockData()

const currentPage = ref(1)
const pageSize = ref(10)
const loading = ref(false)
let loadingTimer = null

const selectedRowIds = ref([])

const filteredRows = computed(() => {
  let rows = mockData.filter(r => r.type === activeTab.value)
  const f = activeFilters.value
  
  if (f.unit1) rows = rows.filter(r => r.unit1.includes(f.unit1))
  if (f.unit2) rows = rows.filter(r => r.unit2.includes(f.unit2))
  if (f.unit3) rows = rows.filter(r => r.unit3.includes(f.unit3))
  if (f.stationName) rows = rows.filter(r => r.stationName.includes(f.stationName))
  if (f.stationCode) rows = rows.filter(r => r.stationCode.includes(f.stationCode))
  if (f.inboundTime) rows = rows.filter(r => r.inboundTime.includes(f.inboundTime))
  if (f.projectName) rows = rows.filter(r => r.projectName.includes(f.projectName))
  if (f.poNumber) rows = rows.filter(r => r.poNumber.includes(f.poNumber))
  if (f.materialCode) rows = rows.filter(r => r.materialCode.includes(f.materialCode))
  if (f.materialDesc) rows = rows.filter(r => r.materialDesc.includes(f.materialDesc))
  
  return rows
})

// 默认按入站时间倒序
const sortState = ref({ key: 'inboundTime', direction: 'desc' })

function sortValue(row, key) {
  return row[key] ?? ''
}

const sortedRows = computed(() => {
  const rows = [...filteredRows.value]
  if (!sortState.value.key) return rows
  const { key, direction } = sortState.value
  
  rows.sort((left, right) => {
    const leftValue = sortValue(left, key)
    const rightValue = sortValue(right, key)
    
    if (leftValue < rightValue) return direction === 'asc' ? -1 : 1
    if (leftValue > rightValue) return direction === 'asc' ? 1 : -1
    return 0
  })
  
  return rows
})

function toggleSort(key) {
  if (sortState.value.key === key) {
    sortState.value = {
      key,
      direction: sortState.value.direction === 'asc' ? 'desc' : 'asc',
    }
  } else {
    sortState.value = { key, direction: 'asc' }
  }
}

function sortIndicator(key) {
  if (sortState.value.key !== key) return '↕'
  return sortState.value.direction === 'asc' ? '↑' : '↓'
}

const pageCount = computed(() => Math.max(1, Math.ceil(sortedRows.value.length / pageSize.value)))
const paginatedRows = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  return sortedRows.value.slice(start, start + pageSize.value)
})
const startIndex = computed(() => (currentPage.value - 1) * pageSize.value)

function applyFilters() {
  activeFilters.value = { ...form }
  currentPage.value = 1
  selectedRowIds.value = []
  startLoading()
}

function resetFilters() {
  Object.assign(form, createEmptyForm())
  activeFilters.value = createEmptyForm()
  currentPage.value = 1
  selectedRowIds.value = []
  startLoading()
}

function startLoading() {
  loading.value = true
  window.clearTimeout(loadingTimer)
  loadingTimer = window.setTimeout(() => {
    loading.value = false
  }, 400)
}

function gotoPage(page) {
  currentPage.value = Math.min(Math.max(1, page), pageCount.value)
}

const columnPickerOpen = ref(false)
const columnPickerRef = ref(null)

const isAllColumnsSelected = computed(() => {
  return tableColumns.value.every(col => col.visible || col.key === 'selection' || col.key === 'index' || col.key === 'actions')
})

const isIndeterminate = computed(() => {
  const visibleCount = tableColumns.value.filter(col => col.visible && col.key !== 'selection' && col.key !== 'index' && col.key !== 'actions').length
  const totalCount = tableColumns.value.length - 3
  return visibleCount > 0 && visibleCount < totalCount
})

function toggleColumnPicker() {
  columnPickerOpen.value = !columnPickerOpen.value
}

function toggleAllColumns(e) {
  const checked = e.target.checked
  tableColumns.value.forEach(col => {
    if (col.key !== 'selection' && col.key !== 'index' && col.key !== 'actions') {
      col.visible = checked
    }
  })
}

const isAllRowsSelected = computed(() => {
  return paginatedRows.value.length > 0 && paginatedRows.value.every(row => selectedRowIds.value.includes(row.id))
})

const isSomeRowsSelected = computed(() => {
  const selectedCount = paginatedRows.value.filter(row => selectedRowIds.value.includes(row.id)).length
  return selectedCount > 0 && selectedCount < paginatedRows.value.length
})

function toggleAllRows(e) {
  const checked = e.target.checked
  if (checked) {
    const newSelected = new Set([...selectedRowIds.value, ...paginatedRows.value.map(r => r.id)])
    selectedRowIds.value = Array.from(newSelected)
  } else {
    selectedRowIds.value = selectedRowIds.value.filter(id => !paginatedRows.value.find(r => r.id === id))
  }
}

function toggleRow(row) {
  const index = selectedRowIds.value.indexOf(row.id)
  if (index === -1) {
    selectedRowIds.value.push(row.id)
  } else {
    selectedRowIds.value.splice(index, 1)
  }
}

function handleTabChange(tab) {
  activeTab.value = tab
  currentPage.value = 1
  selectedRowIds.value = []
  startLoading()
}

onMounted(() => {
  document.addEventListener('click', (e) => {
    if (columnPickerRef.value && !columnPickerRef.value.contains(e.target)) {
      columnPickerOpen.value = false
    }
  })
  startLoading()
})

onBeforeUnmount(() => {
  window.clearTimeout(loadingTimer)
})

function handleExport() {
  alert('导出功能开发中')
}

function handleDetailLog(row) {
  alert(`查看明细流水：${row.materialCode}`)
}
</script>

<template>
  <div style="display: contents;">
    <section class="panel panel--filters">
      <form class="filter-grid" @submit.prevent="applyFilters">
        <label class="field">
          <span>一级单位</span>
          <input v-model.trim="form.unit1" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>二级单位/部门</span>
          <input v-model.trim="form.unit2" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>三级部门</span>
          <input v-model.trim="form.unit3" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>材料站全称</span>
          <input v-model.trim="form.stationName" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>材料站编码</span>
          <input v-model.trim="form.stationCode" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>入站时间</span>
          <input v-model.trim="form.inboundTime" type="date" placeholder="请选择" />
        </label>
        <label class="field">
          <span>项目名称</span>
          <input v-model.trim="form.projectName" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>采购订单编号</span>
          <input v-model.trim="form.poNumber" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>物料编码</span>
          <input v-model.trim="form.materialCode" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>物料描述</span>
          <input v-model.trim="form.materialDesc" type="text" placeholder="请输入" />
        </label>

        <div class="filter-actions" style="grid-column: span 12;">
          <button class="btn btn--primary" type="submit">
            <span class="btn__icon btn__icon--light" aria-hidden="true">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <circle cx="11" cy="11" r="6.5"></circle>
                <path d="M16.5 16.5L21 21"></path>
              </svg>
            </span>
            查询
          </button>
          <button class="btn btn--ghost" type="button" @click="resetFilters">
            <span class="btn__icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M4 12a8 8 0 1 0 2.34-5.66"></path>
                <path d="M4 4v5h5"></path>
              </svg>
            </span>
            重置
          </button>
        </div>
      </form>
    </section>

    <section class="panel panel--results">
      <!-- Toolbar -->
      <div style="display: flex; justify-content: space-between; align-items: center; padding: 12px 20px; border-bottom: 1px solid var(--line);">
        <div class="tabs-container">
          <button 
            class="tab-btn" 
            :class="{ 'tab-btn--active': activeTab === 'stock' }" 
            @click="handleTabChange('stock')"
          >
            存量物资
          </button>
          <button 
            class="tab-btn" 
            :class="{ 'tab-btn--active': activeTab === 'incremental' }" 
            @click="handleTabChange('incremental')"
          >
            增量物资
          </button>
        </div>
        
        <div class="toolbar">
          <button class="btn btn--ghost" type="button" @click="handleExport">
            <span class="btn__icon">↓</span>
            导出
          </button>
          <div ref="columnPickerRef" class="column-picker">
            <button
              class="btn btn--ghost column-picker__trigger"
              type="button"
              :aria-expanded="columnPickerOpen"
              aria-label="列筛选"
              @click="toggleColumnPicker"
            >
              <span class="btn__icon btn__icon--columns" aria-hidden="true">
                <svg viewBox="0 0 24 24" aria-hidden="true">
                  <path d="M4 6h4v12H4z"></path>
                  <path d="M10 6h4v12h-4z"></path>
                  <path d="M16 6h4v12h-4z"></path>
                </svg>
              </span>
            </button>
            <div v-show="columnPickerOpen" class="column-picker__dropdown" style="position: absolute; top: calc(100% + 10px); right: 0; z-index: 20; width: 260px; padding: 12px; border-radius: 14px; border: 1px solid var(--line); background: rgba(255, 255, 255, 0.98); box-shadow: var(--shadow-md);">
              <div class="column-picker__title" style="padding-bottom: 8px; color: var(--subtext); font-size: 12px; font-weight: 600;">列筛选</div>
              <div class="column-picker__header">
                <label class="column-picker__option">
                  <input
                    type="checkbox"
                    :checked="isAllColumnsSelected"
                    :indeterminate.prop="isIndeterminate"
                    class="table-checkbox"
                    @change="toggleAllColumns"
                  >
                  <span>全选</span>
                </label>
              </div>
              <div class="column-picker__list">
                <label
                  v-for="col in tableColumns"
                  :key="col.key"
                  class="column-picker__option"
                >
                  <input
                    v-model="col.visible"
                    type="checkbox"
                    class="table-checkbox"
                    :disabled="col.key === 'selection' || col.key === 'index' || col.key === 'actions'"
                  >
                  <span>{{ col.label }}</span>
                </label>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <div class="table-wrap" style="overflow-x: auto; position: relative;">
        <table class="data-table" style="min-width: 2500px; table-layout: fixed; border-collapse: collapse;">
          <thead>
            <tr>
              <th
                v-for="column in tableColumns.filter(c => c.visible !== false)"
                :key="column.key"
                :style="{ 
                  minWidth: column.width, width: column.width, maxWidth: column.width, boxSizing: 'border-box', 
                  textAlign: (column.key === 'selection' || column.key === 'actions') ? 'center' : 'left' 
                }"
                :class="{
                  'cell--actions-header': column.key === 'actions',
                  'cell--checkbox': column.key === 'selection',
                  'is-sortable': column.sortable,
                  'is-sorted': sortState.key === column.key
                }"
                @click="column.sortable && toggleSort(column.key)"
              >
                <template v-if="column.key === 'selection'">
                  <input 
                    type="checkbox" 
                    class="table-checkbox"
                    :checked="isAllRowsSelected"
                    :indeterminate.prop="isSomeRowsSelected"
                    @change="toggleAllRows"
                  />
                </template>
                <template v-else>
                  <span>{{ column.label }}</span>
                  <span v-if="column.sortable" class="sort-mark">{{ sortIndicator(column.key) }}</span>
                </template>
              </th>
            </tr>
          </thead>
          <tbody v-if="loading">
            <tr v-for="skeleton in pageSize" :key="skeleton" class="skeleton-row">
              <td 
                v-for="column in tableColumns.filter(c => c.visible !== false)" 
                :key="column.key" 
                :class="{ 'cell--actions': column.key === 'actions' }" 
                :style="column.key === 'actions' ? { minWidth: column.width, width: column.width, maxWidth: column.width, boxSizing: 'border-box' } : {}"
              >
                <span class="skeleton-line"></span>
              </td>
            </tr>
          </tbody>
          <tbody v-else-if="paginatedRows.length">
            <tr v-for="(row, index) in paginatedRows" :key="row.id">
              <template v-for="column in tableColumns.filter(c => c.visible !== false)" :key="column.key">
                <td v-if="column.key === 'selection'" class="cell--checkbox" style="text-align: center;">
                  <input 
                    type="checkbox" 
                    class="table-checkbox"
                    :checked="selectedRowIds.includes(row.id)"
                    @change="toggleRow(row)"
                  />
                </td>
                <td v-else-if="column.key === 'index'" class="cell--index">
                  {{ startIndex + index + 1 }}
                </td>
                <td v-else-if="column.key === 'actions'" class="actions cell--actions" :style="{ width: column.width, minWidth: column.width, maxWidth: column.width, boxSizing: 'border-box', whiteSpace: 'nowrap', textAlign: 'center' }">
                  <a href="javascript:void(0)" class="text-link" @click="handleDetailLog(row)">明细流水</a>
                </td>
                <td v-else :title="row[column.key]" style="white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">
                  {{ row[column.key] || '-' }}
                </td>
              </template>
            </tr>
          </tbody>
          <tbody v-else>
            <tr>
              <td :colspan="tableColumns.filter(c => c.visible !== false).length">
                <div class="empty-state">
                  <div class="empty-state__icon">□</div>
                  <strong>暂无匹配数据</strong>
                  <p>请调整查询条件后重新检索。</p>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="table-footer">
        <div class="page-size">
          <span style="margin-right: 8px;">共 {{ filteredRows.length }} 条</span>
          每页
          <select v-model.number="pageSize" @change="gotoPage(1)">
            <option :value="10">10 条</option>
            <option :value="20">20 条</option>
            <option :value="30">30 条</option>
          </select>
        </div>
        <div class="pagination">
          <button class="page-btn" type="button" :disabled="currentPage === 1" @click="gotoPage(currentPage - 1)">‹</button>
          <button
            v-for="page in pageCount"
            :key="page"
            class="page-btn"
            :class="{ 'page-btn--active': currentPage === page }"
            type="button"
            @click="gotoPage(page)"
          >
            {{ page }}
          </button>
          <button class="page-btn" type="button" :disabled="currentPage === pageCount" @click="gotoPage(currentPage + 1)">›</button>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.text-link {
  color: #1e3a8a;
  text-decoration: none;
  cursor: pointer;
}
.text-link:hover {
  text-decoration: underline;
}

.tabs-container {
  display: flex;
  gap: 16px;
}

.tab-btn {
  background: transparent;
  border: none;
  padding: 8px 12px;
  font-size: 14px;
  font-weight: 500;
  color: var(--text-secondary);
  cursor: pointer;
  position: relative;
}

.tab-btn--active {
  color: var(--primary);
}

.tab-btn--active::after {
  content: '';
  position: absolute;
  bottom: -13px; /* align with border-bottom of the parent container */
  left: 0;
  right: 0;
  height: 2px;
  background-color: var(--primary);
}

/* 冻结操作列在最右侧 */
.cell--actions-header,
.cell--actions {
  position: sticky;
  right: 0;
  background-color: #fff;
  z-index: 10;
  border-left: 1px solid var(--line);
}
.cell--actions-header {
  background-color: var(--bg-alt); /* match thead bg */
}
</style>
