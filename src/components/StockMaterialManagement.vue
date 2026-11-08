<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const createEmptyForm = () => ({
  unitName: '',
  poNumber: '',
  materialCode: '',
  materialDesc: '',
  projectName: '',
  materialNature: '',
  stationName: '',
  stationCode: '',
  disposalStatus: '',
  dataSource: ''
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())
const filtersExpanded = ref(false)

const materialNatureOptions = ['工程物资', '结余物资', '备品备件']
const disposalStatusOptions = ['未处置', '处置中', '已处置']
const dataSourceOptions = ['e收货', 'pc端导入']
const unitNameOptions = ['国网沈阳供电公司', '国网锦州供电公司', '国网葫芦岛供电公司', '国网大连供电公司', '国网辽阳供电公司']

const tableColumns = ref([
  { key: 'selection', label: '', width: '40px', sortable: false, visible: true },
  { key: 'index', label: '序号', width: '60px', sortable: false, visible: true },
  { key: 'warning', label: '预警', width: '80px', sortable: true, visible: true },
  { key: 'unitName', label: '单位名称', width: '150px', sortable: true, visible: true },
  { key: 'professionalDept', label: '专业部门', width: '120px', sortable: true, visible: true },
  { key: 'professionalSubdiv', label: '专业细分', width: '120px', sortable: true, visible: true },
  { key: 'assetAttribute', label: '资产属性', width: '120px', sortable: true, visible: true },
  { key: 'materialCode', label: '物料编码', width: '120px', sortable: true, visible: true },
  { key: 'materialDesc', label: '物料描述', width: '180px', sortable: true, visible: true },
  { key: 'materialNature', label: '物资性质', width: '100px', sortable: true, visible: true },
  { key: 'stationName', label: '材料站全称', width: '180px', sortable: true, visible: true },
  { key: 'stationCode', label: '材料站编码', width: '120px', sortable: true, visible: true },
  { key: 'storageLocationNature', label: '存放地点性质', width: '120px', sortable: true, visible: true },
  { key: 'currentStorageLocation', label: '现存放地点', width: '150px', sortable: true, visible: true },
  { key: 'confirmedStorageLocation', label: '确定存储地点', width: '150px', sortable: true, visible: true },
  { key: 'projectName', label: '项目名称', width: '160px', sortable: true, visible: true },
  { key: 'wbsCode', label: 'WBS编码', width: '120px', sortable: true, visible: true },
  { key: 'poNumber', label: '采购订单编号', width: '140px', sortable: true, visible: true },
  { key: 'importQty', label: '导入数量', width: '100px', sortable: true, visible: true },
  { key: 'unit', label: '计量单位', width: '80px', sortable: true, visible: true },
  { key: 'unitPrice', label: '单价/元', width: '100px', sortable: true, visible: true },
  { key: 'totalPrice', label: '总价/元', width: '100px', sortable: true, visible: true },
  { key: 'remainingQty', label: '剩余数量', width: '100px', sortable: true, visible: true },
  { key: 'plannedDisposalMethod', label: '计划处置方式', width: '120px', sortable: true, visible: true },
  { key: 'plannedDisposalTime', label: '计划处置时间', width: '120px', sortable: true, visible: true },
  { key: 'importTime', label: '导入时间', width: '150px', sortable: true, visible: true },
  { key: 'disposalUpdateTime', label: '处置更新时间', width: '150px', sortable: true, visible: true },
  { key: 'disposalStatus', label: '处置状态', width: '100px', sortable: true, visible: true },
  { key: 'formationReason', label: '形成原因', width: '120px', sortable: true, visible: true },
  { key: 'remark', label: '备注', width: '150px', sortable: true, visible: true },
  { key: 'importer', label: '导入人', width: '100px', sortable: true, visible: true },
  { key: 'actions', label: '操作', width: '160px', sortable: false, visible: true },
])

const generateMockData = () => {
  const cityDistrictMap = {
    '沈阳': '和平区',
    '锦州': '古塔区',
    '葫芦岛': '连山区',
    '大连': '中山区',
    '辽阳': '白塔区'
  }

  return Array.from({ length: 25 }, (_, i) => {
    const unitName = unitNameOptions[i % unitNameOptions.length]
    const city = unitName.replace('国网', '').replace('供电公司', '')
    const district = cityDistrictMap[city] || '市区'
    const stationName = `${city}${district}综合材料站`

    return {
      id: i + 1,
      warning: i % 3 === 0 ? 'red' : (i % 3 === 1 ? 'orange' : 'green'),
      unitName,
      professionalDept: '物资部',
      professionalSubdiv: '仓储组',
      assetAttribute: '固定资产',
      materialCode: `MAT${String(i + 1).padStart(5, '0')}`,
      materialDesc: `10kV电力电缆型号${i}`,
      materialNature: materialNatureOptions[i % 3],
      stationName,
      stationCode: `STA${String((i % 2) + 1).padStart(3, '0')}`,
      storageLocationNature: '室内库房',
      currentStorageLocation: 'A区-01架',
      confirmedStorageLocation: 'A区-01架',
      projectName: '电网改造升级项目' + (i % 3 === 0 ? '一期' : '二期'),
      wbsCode: `WBS-2026-${String(i + 1).padStart(3, '0')}`,
      poNumber: `PO202611${String(i + 1).padStart(4, '0')}`,
      importQty: (i + 1) * 10,
      unit: '米',
      unitPrice: 150.00,
      totalPrice: (i + 1) * 1500.00,
      remainingQty: (i + 1) * 5,
      plannedDisposalMethod: '退库',
      plannedDisposalTime: `2026-12-${String((i % 28) + 1).padStart(2, '0')}`,
      importTime: `2026-11-${String((i % 28) + 1).padStart(2, '0')} 14:30`,
      disposalUpdateTime: `2026-11-${String((i % 28) + 2).padStart(2, '0')} 09:15`,
      disposalStatus: disposalStatusOptions[i % 3],
      formationReason: '工程结余',
      remark: '-',
      importer: '张伟',
      dataSource: dataSourceOptions[i % 2]
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
  let rows = mockData
  const f = activeFilters.value
  
  if (f.unitName) rows = rows.filter(r => r.unitName.includes(f.unitName))
  if (f.poNumber) rows = rows.filter(r => r.poNumber.includes(f.poNumber))
  if (f.materialCode) rows = rows.filter(r => r.materialCode.includes(f.materialCode))
  if (f.materialDesc) rows = rows.filter(r => r.materialDesc.includes(f.materialDesc))
  if (f.projectName) rows = rows.filter(r => r.projectName.includes(f.projectName))
  if (f.materialNature) rows = rows.filter(r => r.materialNature === f.materialNature)
  if (f.stationName) rows = rows.filter(r => r.stationName.includes(f.stationName))
  if (f.stationCode) rows = rows.filter(r => r.stationCode.includes(f.stationCode))
  if (f.disposalStatus) rows = rows.filter(r => r.disposalStatus === f.disposalStatus)
  if (f.dataSource) rows = rows.filter(r => r.dataSource === f.dataSource)
  
  return rows
})

const sortState = ref({ key: 'importTime', direction: 'desc' })

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

function toggleFiltersExpanded() {
  filtersExpanded.value = !filtersExpanded.value
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

function handleImport() {
  alert('导入功能开发中')
}

function handleBatchDelete() {
  if (selectedRowIds.value.length === 0) {
    alert('请先选择要删除的数据')
    return
  }
  alert(`批量删除选中的 ${selectedRowIds.value.length} 条数据`)
}

function handleAssociateStation() {
  if (selectedRowIds.value.length === 0) {
    alert('请先选择要关联的数据')
    return
  }
  alert(`关联选中的 ${selectedRowIds.value.length} 条数据到材料站`)
}

function handleAction(action, row) {
  alert(`触发操作：${action}`)
}

</script>

<template>
  <div style="display: contents;">
    <section class="panel panel--filters">
      <form class="filter-grid" :class="{ 'filter-grid--collapsed': !filtersExpanded }" @submit.prevent="applyFilters">
        <label class="field">
          <span>单位名称</span>
          <input v-model.trim="form.unitName" type="text" placeholder="请输入" />
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
        <label class="field" v-show="filtersExpanded">
          <span>项目名称</span>
          <input v-model.trim="form.projectName" type="text" placeholder="请输入" />
        </label>
        <label class="field" v-show="filtersExpanded">
          <span>物资性质</span>
          <select v-model="form.materialNature">
            <option value="">全部</option>
            <option v-for="opt in materialNatureOptions" :key="opt" :value="opt">{{ opt }}</option>
          </select>
        </label>
        <label class="field" v-show="filtersExpanded">
          <span>材料站全称</span>
          <input v-model.trim="form.stationName" type="text" placeholder="请输入" />
        </label>
        <label class="field" v-show="filtersExpanded">
          <span>材料站编码</span>
          <input v-model.trim="form.stationCode" type="text" placeholder="请输入" />
        </label>
        <label class="field" v-show="filtersExpanded">
          <span>处置状态</span>
          <select v-model="form.disposalStatus">
            <option value="">全部</option>
            <option v-for="opt in disposalStatusOptions" :key="opt" :value="opt">{{ opt }}</option>
          </select>
        </label>
        <label class="field" v-show="filtersExpanded">
          <span>数据来源</span>
          <select v-model="form.dataSource">
            <option value="">全部</option>
            <option v-for="opt in dataSourceOptions" :key="opt" :value="opt">{{ opt }}</option>
          </select>
        </label>

        <div class="filter-actions" :style="{ gridColumn: filtersExpanded ? 'span 8' : 'span 4' }">
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
          <button class="filter-toggle" type="button" :aria-expanded="filtersExpanded" @click="toggleFiltersExpanded">
            {{ filtersExpanded ? '收起' : '更多' }}
          </button>
        </div>
      </form>
    </section>

    <section class="panel panel--results">
      <!-- Toolbar -->
      <div style="display: flex; justify-content: flex-end; align-items: center; padding: 12px 20px; border-bottom: 1px solid var(--line);">
        <div class="toolbar">
          <button class="btn btn--primary" type="button" @click="handleAssociateStation">
            关联材料站
          </button>
          <button class="btn btn--ghost" type="button" @click="handleBatchDelete">
            批量删除
          </button>
          <button class="btn btn--ghost" type="button" @click="handleImport">
            <span class="btn__icon">↑</span>
            导入
          </button>
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
      
      <div class="table-wrap">
        <table class="data-table" style="min-width: 3200px; table-layout: fixed;">
          <thead>
            <tr>
              <th
                v-for="column in tableColumns.filter(c => c.visible !== false)"
                :key="column.key"
                :style="{ minWidth: column.width, width: column.width, maxWidth: column.width, boxSizing: 'border-box', textAlign: (column.key === 'selection' || column.key === 'warning') ? 'center' : 'left' }"
                :class="{
                  'cell--actions': column.key === 'actions',
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
              <td v-for="column in tableColumns.filter(c => c.visible !== false)" :key="column.key" :class="{ 'cell--actions': column.key === 'actions' }" :style="column.key === 'actions' ? { minWidth: column.width, width: column.width, maxWidth: column.width, boxSizing: 'border-box' } : {}">
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
                <td v-else-if="column.key === 'actions'" class="actions cell--actions" :style="{ width: column.width, minWidth: column.width, maxWidth: column.width, boxSizing: 'border-box', whiteSpace: 'nowrap' }">
                  <button type="button" class="text-button" @click="handleAction('编辑', row)">编辑</button>
                  <button type="button" class="text-button" @click="handleAction('处置', row)">处置</button>
                  <button type="button" class="text-button" @click="handleAction('记录', row)">记录</button>
                </td>
                <td v-else-if="column.key === 'warning'" style="text-align: center;">
                  <svg v-if="row.warning === 'red'" viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="var(--danger)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="12" cy="12" r="10"></circle>
                    <line x1="12" y1="8" x2="12" y2="12"></line>
                    <line x1="12" y1="16" x2="12.01" y2="16"></line>
                  </svg>
                  <svg v-else-if="row.warning === 'orange'" viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="var(--warning)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path>
                    <line x1="12" y1="9" x2="12" y2="13"></line>
                    <line x1="12" y1="17" x2="12.01" y2="17"></line>
                  </svg>
                  <svg v-else-if="row.warning === 'green'" viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="var(--success)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path>
                    <polyline points="22 4 12 14.01 9 11.01"></polyline>
                  </svg>
                </td>
                <td v-else-if="column.key === 'disposalStatus'">
                  <span class="chip" :class="row.disposalStatus === '已处置' ? 'chip--success' : (row.disposalStatus === '处置中' ? 'chip--warning' : 'chip--neutral')">{{ row.disposalStatus }}</span>
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
.text-button {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 4px 8px;
  color: #1e3a8a;
  text-decoration: none;
  white-space: nowrap;
}
.text-button:hover {
  opacity: 0.8;
}
.text-button--disabled {
  color: var(--text-tertiary);
  cursor: not-allowed;
}
.text-button--disabled:hover {
  opacity: 1;
}
</style>
