<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const activeTab = ref('non-ecommerce')

const createEmptyForm = () => ({
  projectName: '',
  projectUnit: '',
  stationName: '',
  stationCode: '',
  materialCode: '',
  materialDesc: '',
  firstLevelUnit: '',
  applyTimeStart: '',
  applyTimeEnd: '',
  postDateStart: '',
  postDateEnd: ''
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())

const tableColumns = ref([
  { key: 'index', label: '序号', width: '60px', sortable: false, visible: true },
  { key: 'purchaseOrderNo', label: '采购订单编号', width: '140px', sortable: true, visible: true },
  { key: 'lineItem', label: '订单行项目', width: '110px', sortable: true, visible: true },
  { key: 'materialCode', label: '物料编码', width: '120px', sortable: true, visible: true },
  { key: 'materialDesc', label: '物料描述', width: '180px', sortable: true, visible: true },
  { key: 'quantity', label: '数量', width: '80px', sortable: true, visible: true },
  { key: 'unit', label: '计量单位', width: '90px', sortable: true, visible: true },
  { key: 'totalPrice', label: '总价(含税：元)', width: '130px', sortable: true, visible: true },
  { key: 'batchNo', label: '批次号', width: '100px', sortable: true, visible: true },
  { key: 'projectName', label: '项目名称', width: '160px', sortable: true, visible: true },
  { key: 'wbsCode', label: 'WBS编码', width: '120px', sortable: true, visible: true },
  { key: 'projectUnit', label: '项目单位', width: '140px', sortable: true, visible: true },
  { key: 'operator', label: '操作人', width: '90px', sortable: true, visible: true },
  { key: 'operatorPhone', label: '操作人联系方式', width: '130px', sortable: true, visible: true },
  { key: 'stationName', label: '材料站名称', width: '160px', sortable: true, visible: true },
  { key: 'stationCode', label: '材料站编码', width: '120px', sortable: true, visible: true },
  { key: 'stationAddress', label: '材料站地址', width: '180px', sortable: true, visible: true },
  { key: 'firstLevelUnit', label: '一级单位', width: '140px', sortable: true, visible: true },
  { key: 'secondLevelUnit', label: '二级单位/部门', width: '150px', sortable: true, visible: true },
  { key: 'thirdLevelDept', label: '三级部门', width: '120px', sortable: true, visible: true },
  { key: 'orderType', label: '订单类型', width: '100px', sortable: true, visible: true },
  { key: 'receiptDate', label: '收货日期', width: '120px', sortable: true, visible: true },
  { key: 'postDate', label: '过账日期', width: '120px', sortable: true, visible: true },
  { key: 'actions', label: '操作', width: '120px', sortable: false, visible: true },
])

// Mock data generator
const generateMockData = (type) => {
  return Array.from({ length: 25 }, (_, i) => ({
    id: i + 1,
    purchaseOrderNo: 'PO' + String(Math.floor(Math.random() * 10000000)).padStart(7, '0'),
    lineItem: String((i + 1) * 10),
    materialCode: '50' + String(Math.floor(Math.random() * 1000000)).padStart(6, '0'),
    materialDesc: '交流悬式瓷绝缘子 U160BLP' + (i % 3 === 0 ? ' (加长测试数据超长显示测试)' : ''),
    quantity: Math.floor(Math.random() * 100) + 1,
    unit: '片',
    totalPrice: (Math.random() * 10000).toFixed(2),
    batchNo: 'B' + String(Math.floor(Math.random() * 10000)).padStart(4, '0'),
    projectName: '沈阳500kV盛京变电站扩建工程',
    wbsCode: 'WBS-' + String(Math.floor(Math.random() * 1000)).padStart(4, '0'),
    projectUnit: '国网辽宁省电力有限公司',
    operator: '张三',
    operatorPhone: '13800138000',
    stationName: '国网沈阳供电公司铁西物资供应中心',
    stationCode: 'ST-SY-001',
    stationAddress: '辽宁省沈阳市铁西区北二中路18号',
    firstLevelUnit: '国网辽宁省电力有限公司',
    secondLevelUnit: '国网沈阳供电公司',
    thirdLevelDept: '物资部',
    orderType: type === 'ecommerce' ? '电商' : '非电商',
    receiptDate: '2026-10-20',
    postDate: '2026-10-21'
  }))
}

const allData = {
  'non-ecommerce': generateMockData('non-ecommerce'),
  'ecommerce': generateMockData('ecommerce')
}

const currentPage = ref(1)
const pageSize = ref(10)
const loading = ref(false)
let loadingTimer = null

const currentTabData = computed(() => allData[activeTab.value])

// Basic client-side filtering and pagination
const filteredRows = computed(() => {
  let rows = currentTabData.value
  const f = activeFilters.value
  
  if (f.projectName) rows = rows.filter(r => r.projectName.includes(f.projectName))
  if (f.projectUnit) rows = rows.filter(r => r.projectUnit.includes(f.projectUnit))
  if (f.stationName) rows = rows.filter(r => r.stationName.includes(f.stationName))
  if (f.stationCode) rows = rows.filter(r => r.stationCode.includes(f.stationCode))
  if (f.materialCode) rows = rows.filter(r => r.materialCode.includes(f.materialCode))
  if (f.materialDesc) rows = rows.filter(r => r.materialDesc.includes(f.materialDesc))
  if (f.firstLevelUnit) rows = rows.filter(r => r.firstLevelUnit.includes(f.firstLevelUnit))
  if (f.applyTimeStart) rows = rows.filter(r => r.receiptDate >= f.applyTimeStart)
  if (f.applyTimeEnd) rows = rows.filter(r => r.receiptDate <= f.applyTimeEnd)
  if (f.postDateStart) rows = rows.filter(r => r.postDate >= f.postDateStart)
  if (f.postDateEnd) rows = rows.filter(r => r.postDate <= f.postDateEnd)
  
  return rows
})

const sortState = ref({ key: 'receiptDate', direction: 'desc' })

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
  startLoading()
}

function resetFilters() {
  Object.assign(form, createEmptyForm())
  activeFilters.value = createEmptyForm()
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

const isAllColumnsSelected = computed(() => {
  return tableColumns.value.every(col => col.visible || col.key === 'index' || col.key === 'actions')
})

const isIndeterminate = computed(() => {
  const visibleCount = tableColumns.value.filter(col => col.visible && col.key !== 'index' && col.key !== 'actions').length
  const totalCount = tableColumns.value.length - 2
  return visibleCount > 0 && visibleCount < totalCount
})

function toggleColumnPicker() {
  columnPickerOpen.value = !columnPickerOpen.value
}

function toggleAllColumns(e) {
  const checked = e.target.checked
  tableColumns.value.forEach(col => {
    if (col.key !== 'index' && col.key !== 'actions') {
      col.visible = checked
    }
  })
}

// Close column picker when clicking outside
onMounted(() => {
  document.addEventListener('click', (e) => {
    if (columnPickerRef.value && !columnPickerRef.value.contains(e.target)) {
      columnPickerOpen.value = false
    }
  })
})

onBeforeUnmount(() => {
  // Event listener is not explicitly removed in this simple setup since we use an anonymous function,
  // but it's safe enough for this component lifecycle. 
})

function formatTextTruncate(text, maxLength = 10) {
  if (!text) return '-'
  return text.length > maxLength ? text.slice(0, maxLength) + '...' : text
}

function handleExport() {
  alert('导出功能开发中')
}

function handleViewPhoto(row) {
  alert(`查看 ${row.purchaseOrderNo} 现场照片`)
}

onMounted(() => {
  startLoading()
})

onBeforeUnmount(() => {
  window.clearTimeout(loadingTimer)
})
</script>

<template>
  <div style="display: contents;">
    <section class="panel panel--filters">
      <form class="filter-grid filter-grid--collapsed" @submit.prevent="applyFilters">
        <label class="field">
          <span>项目名称</span>
          <input v-model.trim="form.projectName" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>项目单位</span>
          <input v-model.trim="form.projectUnit" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>材料站名称</span>
          <input v-model.trim="form.stationName" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>材料站编码</span>
          <input v-model.trim="form.stationCode" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>物料编码</span>
          <input v-model.trim="form.materialCode" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>物料描述</span>
          <input v-model.trim="form.materialDesc" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>一级单位</span>
          <input v-model.trim="form.firstLevelUnit" type="text" placeholder="请输入" />
        </label>
        
        <label class="field">
          <span>收货日期</span>
          <div class="range-inputs">
            <input v-model="form.applyTimeStart" type="date" />
            <span>-</span>
            <input v-model="form.applyTimeEnd" type="date" />
          </div>
        </label>

        <label class="field">
          <span>过账日期</span>
          <div class="range-inputs">
            <input v-model="form.postDateStart" type="date" />
            <span>-</span>
            <input v-model="form.postDateEnd" type="date" />
          </div>
        </label>

        <div class="filter-actions" style="grid-column: span 18;">
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
      <!-- Tabs and Actions -->
      <div style="display: flex; justify-content: space-between; align-items: flex-end; padding: 0 20px 0; border-bottom: 1px solid var(--line); margin-bottom: 0;">
        <div style="display: flex; gap: 24px;">
          <div 
            v-for="tab in [{id: 'non-ecommerce', label: '非电商订单'}, {id: 'ecommerce', label: '电商订单'}]" 
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

        <div class="toolbar" style="margin-bottom: 12px;">
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
                  v-for="col in tableColumns"
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
        <table class="data-table" style="min-width: 2500px;">
          <thead>
            <tr>
              <th
                v-for="column in tableColumns.filter(c => c.visible !== false)"
                :key="column.key"
                :style="{ minWidth: column.width, width: column.width }"
                :class="[
                  column.key === 'actions' && 'cell--actions',
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
              <td v-for="column in tableColumns.filter(c => c.visible !== false)" :key="column.key" :class="[column.key === 'actions' && 'cell--actions']">
                <span class="skeleton-line"></span>
              </td>
            </tr>
          </tbody>
          <tbody v-else-if="paginatedRows.length">
            <tr v-for="(row, index) in paginatedRows" :key="row.id">
              <template v-for="column in tableColumns.filter(c => c.visible !== false)" :key="column.key">
                <td v-if="column.key === 'index'" class="cell--index">
                  {{ startIndex + index + 1 }}
                </td>
                <td v-else-if="column.key === 'actions'" class="actions cell--actions">
                  <a href="javascript:void(0)" class="text-button" @click="handleViewPhoto(row)" style="color: #1e3a8a; text-decoration: none;">查看现场照片</a>
                </td>
                <td v-else-if="column.key === 'orderType'">
                  <span class="chip chip--soft">{{ row.orderType }}</span>
                </td>
                <td v-else :title="row[column.key]">
                  {{ formatTextTruncate(row[column.key], 15) }}
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
.text-button {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 4px 8px;
}
.text-button:hover {
  opacity: 0.8;
}
</style>
