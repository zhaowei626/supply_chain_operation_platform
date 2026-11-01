<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const activeTab = ref('returnOrder')

const createEmptyReturnOrderForm = () => ({
  returnNo: '',
  stationName: '',
  stationCode: '',
  submitDateStart: '',
  submitDateEnd: '',
  returnReason: '',
  firstLevelUnit: '',
  secondLevelUnit: ''
})

const createEmptyMaterialForm = () => ({
  returnNo: '',
  stationName: '',
  stationCode: '',
  submitDateStart: '',
  submitDateEnd: '',
  consumptionNo: '',
  firstLevelUnit: '',
  secondLevelUnit: '',
  purchaseOrderNo: '',
  materialCode: '',
  materialDesc: '',
  projectName: '',
  orderType: ''
})

const returnOrderForm = reactive(createEmptyReturnOrderForm())
const materialForm = reactive(createEmptyMaterialForm())

const activeReturnOrderFilters = ref(createEmptyReturnOrderForm())
const activeMaterialFilters = ref(createEmptyMaterialForm())

const returnOrderColumns = ref([
  { key: 'index', label: '序号', width: '60px', sortable: false, visible: true },
  { key: 'returnNo', label: '现场退料单号', width: '160px', sortable: true, visible: true },
  { key: 'consumptionNo', label: '关联现场耗用单号', width: '160px', sortable: true, visible: true },
  { key: 'firstLevelUnit', label: '一级单位', width: '140px', sortable: true, visible: true },
  { key: 'secondLevelUnit', label: '二级单位/部门', width: '150px', sortable: true, visible: true },
  { key: 'thirdLevelDept', label: '三级部门', width: '120px', sortable: true, visible: true },
  { key: 'stationName', label: '材料站名称', width: '160px', sortable: true, visible: true },
  { key: 'stationCode', label: '材料站编码', width: '120px', sortable: true, visible: true },
  { key: 'returnReason', label: '退料原因', width: '120px', sortable: true, visible: true },
  { key: 'returnDesc', label: '退料说明', width: '180px', sortable: true, visible: true },
  { key: 'materialCategory', label: '物料品类', width: '120px', sortable: true, visible: true },
  { key: 'operator', label: '操作人', width: '90px', sortable: true, visible: true },
  { key: 'operatorPhone', label: '操作人联系方式', width: '130px', sortable: true, visible: true },
  { key: 'submitTime', label: '提交时间', width: '150px', sortable: true, visible: true },
  { key: 'approver', label: '审批人', width: '90px', sortable: true, visible: true },
  { key: 'approverPhone', label: '审批人联系方式', width: '130px', sortable: true, visible: true },
  { key: 'approvalTime', label: '审批时间', width: '150px', sortable: true, visible: true },
  { key: 'actions', label: '操作', width: '100px', sortable: false, visible: true },
])

const materialColumns = ref([
  { key: 'index', label: '序号', width: '60px', sortable: false, visible: true },
  { key: 'returnNo', label: '现场退料单号', width: '160px', sortable: true, visible: true },
  { key: 'consumptionNo', label: '关联现场耗用单号', width: '160px', sortable: true, visible: true },
  { key: 'purchaseOrderNo', label: '采购订单编号', width: '140px', sortable: true, visible: true },
  { key: 'lineItem', label: '订单行项目', width: '110px', sortable: true, visible: true },
  { key: 'materialCode', label: '物料编码', width: '120px', sortable: true, visible: true },
  { key: 'materialDesc', label: '物料描述', width: '180px', sortable: true, visible: true },
  { key: 'originalQuantity', label: '原领用数量', width: '100px', sortable: true, visible: true },
  { key: 'actualReturnQuantity', label: '实退数量', width: '90px', sortable: true, visible: true },
  { key: 'unit', label: '计量单位', width: '90px', sortable: true, visible: true },
  { key: 'unitPrice', label: '单价(元)', width: '100px', sortable: true, visible: true },
  { key: 'totalPrice', label: '总价(含税：元)', width: '130px', sortable: true, visible: true },
  { key: 'batchNo', label: '批次号', width: '100px', sortable: true, visible: true },
  { key: 'projectName', label: '所属项目', width: '160px', sortable: true, visible: true },
  { key: 'wbsCode', label: 'WBS编码', width: '120px', sortable: true, visible: true },
  { key: 'projectUnit', label: '项目单位', width: '140px', sortable: true, visible: true },
  { key: 'operator', label: '操作人', width: '90px', sortable: true, visible: true },
  { key: 'operatorPhone', label: '操作人联系方式', width: '130px', sortable: true, visible: true },
  { key: 'stationName', label: '材料站名称', width: '160px', sortable: true, visible: true },
  { key: 'stationCode', label: '材料站编码', width: '120px', sortable: true, visible: true },
  { key: 'firstLevelUnit', label: '一级单位', width: '140px', sortable: true, visible: true },
  { key: 'secondLevelUnit', label: '二级单位/部门', width: '150px', sortable: true, visible: true },
  { key: 'thirdLevelDept', label: '三级部门', width: '120px', sortable: true, visible: true },
  { key: 'orderType', label: '订单类型', width: '100px', sortable: true, visible: true },
  { key: 'submitTime', label: '提交时间', width: '150px', sortable: true, visible: true },
])

// Mock data generators
const generateReturnOrderData = () => {
  return Array.from({ length: 25 }, (_, i) => ({
    id: i + 1,
    returnNo: 'XCTL20261101' + String(i + 1).padStart(3, '0'),
    consumptionNo: 'XCHY20261101' + String(i + 1).padStart(3, '0'),
    firstLevelUnit: '国网辽宁省电力有限公司',
    secondLevelUnit: '国网沈阳供电公司',
    thirdLevelDept: '建设部',
    stationName: '国网沈阳供电公司铁西物资供应中心',
    stationCode: 'ST-SY-001',
    returnReason: '设计变更',
    returnDesc: '工程设计变更，部分材料不再需要',
    materialCategory: '线缆类',
    operator: '张三',
    operatorPhone: '13800138000',
    submitTime: '2026-11-01 09:30',
    approver: '李四',
    approverPhone: '13900139000',
    approvalTime: '2026-11-01 10:15',
  }))
}

const generateMaterialData = () => {
  return Array.from({ length: 25 }, (_, i) => ({
    id: i + 1,
    returnNo: 'XCTL20261101' + String(i + 1).padStart(3, '0'),
    consumptionNo: 'XCHY20261101' + String(i + 1).padStart(3, '0'),
    purchaseOrderNo: 'PO' + String(Math.floor(Math.random() * 10000000)).padStart(7, '0'),
    lineItem: String((i + 1) * 10),
    materialCode: '50' + String(Math.floor(Math.random() * 1000000)).padStart(6, '0'),
    materialDesc: '交流悬式瓷绝缘子 U160BLP' + (i % 3 === 0 ? ' (加长测试数据超长显示测试)' : ''),
    originalQuantity: Math.floor(Math.random() * 100) + 50,
    actualReturnQuantity: Math.floor(Math.random() * 50) + 1,
    unit: '片',
    unitPrice: (Math.random() * 100).toFixed(2),
    totalPrice: (Math.random() * 10000).toFixed(2),
    batchNo: 'B' + String(Math.floor(Math.random() * 10000)).padStart(4, '0'),
    projectName: '沈阳500kV盛京变电站扩建工程',
    wbsCode: 'WBS-' + String(Math.floor(Math.random() * 1000)).padStart(4, '0'),
    projectUnit: '国网辽宁省电力有限公司',
    operator: '张三',
    operatorPhone: '13800138000',
    stationName: '国网沈阳供电公司铁西物资供应中心',
    stationCode: 'ST-SY-001',
    firstLevelUnit: '国网辽宁省电力有限公司',
    secondLevelUnit: '国网沈阳供电公司',
    thirdLevelDept: '建设部',
    orderType: i % 2 === 0 ? '电商' : '非电商',
    submitTime: '2026-11-01 09:30',
  }))
}

const allData = {
  returnOrder: generateReturnOrderData(),
  'material-details': generateMaterialData()
}

const currentPage = ref(1)
const pageSize = ref(10)
const loading = ref(false)
let loadingTimer = null

const currentTabData = computed(() => allData[activeTab.value])

// Basic client-side filtering and pagination
const filteredRows = computed(() => {
  let rows = currentTabData.value
  
  if (activeTab.value === 'returnOrder') {
    const f = activeReturnOrderFilters.value
    if (f.stationName) rows = rows.filter(r => r.stationName.includes(f.stationName))
    if (f.stationCode) rows = rows.filter(r => r.stationCode.includes(f.stationCode))
    if (f.returnReason) rows = rows.filter(r => r.returnReason.includes(f.returnReason))
    if (f.firstLevelUnit) rows = rows.filter(r => r.firstLevelUnit.includes(f.firstLevelUnit))
    if (f.secondLevelUnit) rows = rows.filter(r => r.secondLevelUnit.includes(f.secondLevelUnit))
    if (f.returnNo) rows = rows.filter(r => r.returnNo.includes(f.returnNo))
    if (f.submitDateStart) rows = rows.filter(r => r.submitTime.slice(0, 10) >= f.submitDateStart)
    if (f.submitDateEnd) rows = rows.filter(r => r.submitTime.slice(0, 10) <= f.submitDateEnd)
  } else {
    const f = activeMaterialFilters.value
    if (f.returnNo) rows = rows.filter(r => r.returnNo.includes(f.returnNo))
    if (f.consumptionNo) rows = rows.filter(r => r.consumptionNo.includes(f.consumptionNo))
    if (f.stationName) rows = rows.filter(r => r.stationName.includes(f.stationName))
    if (f.stationCode) rows = rows.filter(r => r.stationCode.includes(f.stationCode))
    if (f.firstLevelUnit) rows = rows.filter(r => r.firstLevelUnit.includes(f.firstLevelUnit))
    if (f.secondLevelUnit) rows = rows.filter(r => r.secondLevelUnit.includes(f.secondLevelUnit))
    if (f.submitDateStart) rows = rows.filter(r => r.submitTime.slice(0, 10) >= f.submitDateStart)
    if (f.submitDateEnd) rows = rows.filter(r => r.submitTime.slice(0, 10) <= f.submitDateEnd)
    if (f.purchaseOrderNo) rows = rows.filter(r => r.purchaseOrderNo.includes(f.purchaseOrderNo))
    if (f.materialCode) rows = rows.filter(r => r.materialCode.includes(f.materialCode))
    if (f.materialDesc) rows = rows.filter(r => r.materialDesc.includes(f.materialDesc))
    if (f.projectName) rows = rows.filter(r => r.projectName.includes(f.projectName))
    if (f.orderType) rows = rows.filter(r => r.orderType === f.orderType)
  }
  
  return rows
})

const sortState = ref({ key: 'submitTime', direction: 'desc' })

function sortValue(row, key) {
  const value = row[key]
  if (value === null || value === undefined) return ''
  return String(value).toLowerCase()
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
  if (activeTab.value === 'returnOrder') {
    activeReturnOrderFilters.value = { ...returnOrderForm }
  } else {
    activeMaterialFilters.value = { ...materialForm }
  }
  currentPage.value = 1
  startLoading()
}

function resetFilters() {
  if (activeTab.value === 'returnOrder') {
    Object.assign(returnOrderForm, createEmptyReturnOrderForm())
    activeReturnOrderFilters.value = createEmptyReturnOrderForm()
  } else {
    Object.assign(materialForm, createEmptyMaterialForm())
    activeMaterialFilters.value = createEmptyMaterialForm()
  }
  currentPage.value = 1
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

function handleTabChange(tab) {
  activeTab.value = tab
  currentPage.value = 1
  startLoading()
}

const columnPickerOpen = ref(false)
const columnPickerRef = ref(null)

const currentColumns = computed(() => activeTab.value === 'returnOrder' ? returnOrderColumns.value : materialColumns.value)

const isAllColumnsSelected = computed(() => {
  return currentColumns.value.every(col => col.visible || col.key === 'index' || col.key === 'actions')
})

const isIndeterminate = computed(() => {
  const visibleCount = currentColumns.value.filter(col => col.visible && col.key !== 'index' && col.key !== 'actions').length
  const totalCount = currentColumns.value.length - (activeTab.value === 'returnOrder' ? 2 : 1) // material has no actions
  return visibleCount > 0 && visibleCount < totalCount
})

function toggleColumnPicker() {
  columnPickerOpen.value = !columnPickerOpen.value
}

function toggleAllColumns(e) {
  const checked = e.target.checked
  currentColumns.value.forEach(col => {
    if (col.key !== 'index' && col.key !== 'actions') {
      col.visible = checked
    }
  })
}

onMounted(() => {
  document.addEventListener('click', (e) => {
    if (columnPickerRef.value && !columnPickerRef.value.contains(e.target)) {
      columnPickerOpen.value = false
    }
  })
})

onBeforeUnmount(() => {
  window.clearTimeout(loadingTimer)
})

function formatTextTruncate(text, maxLength = 10) {
  if (!text) return '-'
  return text.length > maxLength ? text.slice(0, maxLength) + '...' : text
}

function handleExport() {
  alert('导出功能开发中')
}

function handleViewMaterialDetails(row) {
  alert(`查看 ${row.returnNo} 物料明细`)
}

onMounted(() => {
  startLoading()
})
</script>

<template>
  <div style="display: contents;">
    <section class="panel panel--filters">
      <!-- Tabs moved to filter area -->
      <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--line); margin-bottom: 16px; padding: 0 20px;">
        <div style="display: flex; gap: 24px;">
          <div 
            v-for="tab in [{id: 'returnOrder', label: '退料单'}, {id: 'material-details', label: '物料明细'}]" 
            :key="tab.id"
            @click="handleTabChange(tab.id)"
            :style="{
              padding: '16px 0 12px 0', 
              cursor: 'pointer', 
              borderBottom: activeTab === tab.id ? '2px solid var(--primary)' : '2px solid transparent',
              color: activeTab === tab.id ? 'var(--primary)' : 'var(--text-secondary)',
              fontWeight: activeTab === tab.id ? '600' : '400',
              transition: 'all 0.2s',
              marginBottom: '-1px'
            }"
          >
            {{ tab.label }}
          </div>
        </div>
        <div style="color: var(--subtext); font-size: 13px; display: flex; align-items: center; gap: 4px;">
          <svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="10"></circle>
            <line x1="12" y1="16" x2="12" y2="12"></line>
            <line x1="12" y1="8" x2="12.01" y2="8"></line>
          </svg>
          本页面展示的数据均为已审批通过的数据
        </div>
      </div>

      <!-- 退料单查询区域 -->
      <form v-if="activeTab === 'returnOrder'" key="returnOrder-form" class="filter-grid filter-grid--collapsed" @submit.prevent="applyFilters">
        <label class="field">
          <span>现场退料单号</span>
          <input v-model.trim="returnOrderForm.returnNo" type="text" placeholder="请输入现场退料单号" />
        </label>
        <label class="field">
          <span>材料站名称</span>
          <input v-model.trim="returnOrderForm.stationName" type="text" placeholder="请输入材料站名称" />
        </label>
        <label class="field">
          <span>材料站编码</span>
          <input v-model.trim="returnOrderForm.stationCode" type="text" placeholder="请输入材料站编码" />
        </label>
        <label class="field">
          <span>提交日期</span>
          <div class="range-inputs">
            <input v-model="returnOrderForm.submitDateStart" type="date" />
            <span>-</span>
            <input v-model="returnOrderForm.submitDateEnd" type="date" />
          </div>
        </label>
        <label class="field">
          <span>退料原因</span>
          <input v-model.trim="returnOrderForm.returnReason" type="text" placeholder="请输入退料原因" />
        </label>
        <label class="field">
          <span>一级单位</span>
          <input v-model.trim="returnOrderForm.firstLevelUnit" type="text" placeholder="请输入一级单位" />
        </label>
        <label class="field">
          <span>二级单位/部门</span>
          <input v-model.trim="returnOrderForm.secondLevelUnit" type="text" placeholder="请输入二级单位/部门" />
        </label>

        <div class="filter-actions" style="grid-column: -7 / -1;">
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

      <!-- 物料明细查询区域 -->
      <form v-if="activeTab === 'material-details'" key="material-form" class="filter-grid filter-grid--collapsed" @submit.prevent="applyFilters">
        <label class="field">
          <span>现场退料单号</span>
          <input v-model.trim="materialForm.returnNo" type="text" placeholder="请输入现场退料单号" />
        </label>
        <label class="field">
          <span>材料站名称</span>
          <input v-model.trim="materialForm.stationName" type="text" placeholder="请输入材料站名称" />
        </label>
        <label class="field">
          <span>材料站编码</span>
          <input v-model.trim="materialForm.stationCode" type="text" placeholder="请输入材料站编码" />
        </label>
        <label class="field">
          <span>提交日期</span>
          <div class="range-inputs">
            <input v-model="materialForm.submitDateStart" type="date" />
            <span>-</span>
            <input v-model="materialForm.submitDateEnd" type="date" />
          </div>
        </label>
        <label class="field">
          <span>关联现场耗用单号</span>
          <input v-model.trim="materialForm.consumptionNo" type="text" placeholder="请输入关联现场耗用单号" />
        </label>
        <label class="field">
          <span>一级单位</span>
          <input v-model.trim="materialForm.firstLevelUnit" type="text" placeholder="请输入一级单位" />
        </label>
        <label class="field">
          <span>二级单位/部门</span>
          <input v-model.trim="materialForm.secondLevelUnit" type="text" placeholder="请输入二级单位/部门" />
        </label>
        <label class="field">
          <span>采购订单编号</span>
          <input v-model.trim="materialForm.purchaseOrderNo" type="text" placeholder="请输入采购订单编号" />
        </label>
        <label class="field">
          <span>物料编码</span>
          <input v-model.trim="materialForm.materialCode" type="text" placeholder="请输入物料编码" />
        </label>
        <label class="field">
          <span>物料描述</span>
          <input v-model.trim="materialForm.materialDesc" type="text" placeholder="请输入物料描述" />
        </label>
        <label class="field">
          <span>所属项目</span>
          <input v-model.trim="materialForm.projectName" type="text" placeholder="请输入所属项目" />
        </label>
        <label class="field">
          <span>订单类型</span>
          <select v-model="materialForm.orderType">
            <option value="">请选择订单类型</option>
            <option value="电商">电商</option>
            <option value="非电商">非电商</option>
          </select>
        </label>

        <div class="filter-actions" style="grid-column: -7 / -1;">
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
      <!-- Toolbar only -->
      <div class="results-bar">
        <div class="toolbar">
          <!-- 退料单没有按钮, 物料明细有导出、列筛选 -->
          <button v-show="activeTab === 'material-details'" class="btn btn--ghost" type="button" @click="handleExport">
            <span class="btn__icon">↓</span>
            导出
          </button>
          <div v-show="activeTab === 'material-details'" ref="columnPickerRef" class="column-picker">
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
            <div v-show="columnPickerOpen" class="column-picker__dropdown">
              <div class="column-picker__header">
                <label class="column-picker__item">
                  <input
                    type="checkbox"
                    :checked="isAllColumnsSelected"
                    :indeterminate="isIndeterminate"
                    class="checkbox"
                    @change="toggleAllColumns"
                  >
                  <span>全选</span>
                </label>
              </div>
              <div class="column-picker__list">
                <label
                  v-for="col in currentColumns"
                  :key="col.key"
                  class="column-picker__item"
                >
                  <input
                    v-model="col.visible"
                    type="checkbox"
                    class="checkbox"
                    :disabled="col.key === 'index' || col.key === 'actions'"
                  >
                  <span>{{ col.label }}</span>
                </label>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <div class="table-wrap">
        <table class="data-table" :style="{ minWidth: activeTab === 'returnOrder' ? '2100px' : '2600px' }">
          <thead>
            <tr>
              <th
                v-for="column in currentColumns.filter(c => c.visible !== false)"
                :key="column.key"
                :style="{ minWidth: column.width, width: column.width }"
                :class="[
                  column.key === 'actions' && 'cell--actions sticky-right',
                  column.sortable && 'is-sortable',
                  sortState.key === column.key && 'is-sorted',
                ]"
                @click="column.sortable && toggleSort(column.key)"
              >
                <span>{{ column.label }}</span>
                <span v-if="column.sortable" class="sort-mark">{{ sortIndicator(column.key) }}</span>
              </th>
            </tr>
          </thead>
          <tbody v-if="loading">
            <tr v-for="skeleton in pageSize" :key="skeleton" class="skeleton-row">
              <td v-for="column in currentColumns.filter(c => c.visible !== false)" :key="column.key" :class="[column.key === 'actions' && 'cell--actions sticky-right']">
                <span class="skeleton-line"></span>
              </td>
            </tr>
          </tbody>
          <tbody v-else-if="paginatedRows.length">
            <tr v-for="(row, index) in paginatedRows" :key="row.id">
              <template v-for="column in currentColumns.filter(c => c.visible !== false)" :key="column.key">
                <td v-if="column.key === 'index'" class="cell--index">
                  {{ startIndex + index + 1 }}
                </td>
                <td v-else-if="column.key === 'actions'" class="actions cell--actions sticky-right" style="background-color: #fff; z-index: 1;">
                  <a href="javascript:void(0)" class="text-button" @click="handleViewMaterialDetails(row)" style="color: #1e3a8a; text-decoration: none;">物料明细</a>
                </td>
                <td v-else :title="row[column.key]">
                  {{ formatTextTruncate(row[column.key], 10) }}
                </td>
              </template>
            </tr>
          </tbody>
          <tbody v-else>
            <tr>
              <td :colspan="currentColumns.filter(c => c.visible !== false).length">
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
.text-button {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 4px 8px;
}
.text-button:hover {
  opacity: 0.8;
}
.sticky-right {
  position: sticky;
  right: 0;
  box-shadow: -2px 0 5px rgba(0, 0, 0, 0.05);
}
</style>
