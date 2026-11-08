<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const createEmptyForm = () => ({
  stationName: '',
  checkName: '',
  checker: '',
  signer: '',
  firstLevelUnit: '',
  secondLevelUnit: '',
  thirdLevelDept: '',
  status: '',
  completionDateStart: '',
  completionDateEnd: '',
  project: ''
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())

const tableColumns = ref([
  { key: 'index', label: '序号', width: '60px', sortable: false, visible: true },
  { key: 'firstLevelUnit', label: '一级单位', width: '140px', sortable: true, visible: true },
  { key: 'secondLevelUnit', label: '二级单位/部门', width: '150px', sortable: true, visible: true },
  { key: 'thirdLevelDept', label: '三级部门', width: '120px', sortable: true, visible: true },
  { key: 'stationName', label: '材料站全称', width: '180px', sortable: true, visible: true },
  { key: 'checkName', label: '盘点名称', width: '200px', sortable: true, visible: true },
  { key: 'completionDate', label: '盘点完成日期', width: '160px', sortable: true, visible: true },
  { key: 'auditSignDate', label: '审核签署日期', width: '160px', sortable: true, visible: true },
  { key: 'project', label: '关联项目', width: '160px', sortable: true, visible: true },
  { key: 'status', label: '盘点状态', width: '100px', sortable: true, visible: true },
  { key: 'checker', label: '盘点人', width: '100px', sortable: true, visible: true },
  { key: 'signer', label: '签署人', width: '100px', sortable: true, visible: true },
  { key: 'actions', label: '操作', width: '280px', sortable: false, visible: true },
])

const generateMockData = () => {
  const names = ['张伟', '王芳', '李娜', '刘强', '陈静']
  const signers = ['赵总', '李工', '王总', '孙工', '-']
  return Array.from({ length: 25 }, (_, i) => {
    const isCompleted = i % 3 !== 0
    return {
      id: i + 1,
      firstLevelUnit: '国网辽宁省电力有限公司',
      secondLevelUnit: i % 2 === 0 ? '国网沈阳供电公司' : '国网大连供电公司',
      thirdLevelDept: '物资部',
      stationName: i % 2 === 0 ? '沈阳和平区综合材料站' : '大连中山区综合材料站',
      checkName: i % 2 === 0 ? '沈阳和平区综合材料站-2026年11月-第1次盘点' : '大连中山区综合材料站-2026年11月-第2次盘点',
      completionDate: `2026-11-${String((i % 28) + 1).padStart(2, '0')} 14:30`,
      auditSignDate: isCompleted ? `2026-11-${String((i % 28) + 2).padStart(2, '0')} 09:15` : '-',
      project: '电网改造升级项目' + (i % 3 === 0 ? '一期' : '二期'),
      status: isCompleted ? '已完成' : '待签署',
      checker: names[i % names.length],
      signer: isCompleted ? signers[i % (signers.length - 1)] : '-'
    }
  })
}

const mockData = generateMockData()

const currentPage = ref(1)
const pageSize = ref(10)
const loading = ref(false)
let loadingTimer = null

const filteredRows = computed(() => {
  let rows = mockData
  const f = activeFilters.value
  
  if (f.stationName) rows = rows.filter(r => r.stationName.includes(f.stationName))
  if (f.checkName) rows = rows.filter(r => r.checkName.includes(f.checkName))
  if (f.checker) rows = rows.filter(r => r.checker.includes(f.checker))
  if (f.signer) rows = rows.filter(r => r.signer.includes(f.signer))
  if (f.firstLevelUnit) rows = rows.filter(r => r.firstLevelUnit.includes(f.firstLevelUnit))
  if (f.secondLevelUnit) rows = rows.filter(r => r.secondLevelUnit.includes(f.secondLevelUnit))
  if (f.thirdLevelDept) rows = rows.filter(r => r.thirdLevelDept.includes(f.thirdLevelDept))
  if (f.status) rows = rows.filter(r => r.status === f.status)
  if (f.project) rows = rows.filter(r => r.project.includes(f.project))
  
  if (f.completionDateStart) rows = rows.filter(r => r.completionDate.slice(0, 10) >= f.completionDateStart)
  if (f.completionDateEnd) rows = rows.filter(r => r.completionDate.slice(0, 10) <= f.completionDateEnd)
  
  return rows
})

const sortState = ref({ key: 'completionDate', direction: 'desc' })

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

function handleAction(action, row) {
  if (action === 'report' && row.status !== '已完成') return
  alert(`触发操作：${action}`)
}

</script>

<template>
  <div style="display: contents;">
    <section class="panel panel--filters">
      <form class="filter-grid filter-grid--collapsed" @submit.prevent="applyFilters">
        <label class="field">
          <span>材料站全称</span>
          <input v-model.trim="form.stationName" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>盘点名称</span>
          <input v-model.trim="form.checkName" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>盘点人</span>
          <input v-model.trim="form.checker" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>签署人</span>
          <input v-model.trim="form.signer" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>一级单位</span>
          <input v-model.trim="form.firstLevelUnit" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>二级单位/部门</span>
          <input v-model.trim="form.secondLevelUnit" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>三级部门</span>
          <input v-model.trim="form.thirdLevelDept" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>盘点状态</span>
          <select v-model="form.status">
            <option value="">全部</option>
            <option value="待签署">待签署</option>
            <option value="已完成">已完成</option>
          </select>
        </label>
        <label class="field">
          <span>关联项目</span>
          <input v-model.trim="form.project" type="text" placeholder="请输入" />
        </label>
        <label class="field">
          <span>盘点完成日期</span>
          <div class="range-inputs">
            <input v-model="form.completionDateStart" type="date" />
            <span>-</span>
            <input v-model="form.completionDateEnd" type="date" />
          </div>
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
      <div style="display: flex; justify-content: flex-end; align-items: center; padding: 12px 20px; border-bottom: 1px solid var(--line);">
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
        <table class="data-table" style="min-width: 1800px; table-layout: fixed;">
          <thead>
            <tr>
              <th
                v-for="column in tableColumns.filter(c => c.visible !== false)"
                :key="column.key"
                :style="{ minWidth: column.width, width: column.width, maxWidth: column.width, boxSizing: 'border-box' }"
                :class="{
                  'cell--actions': column.key === 'actions',
                  'is-sortable': column.sortable,
                  'is-sorted': sortState.key === column.key
                }"
                @click="column.sortable && toggleSort(column.key)"
              >
                <span>{{ column.label }}</span>
                <span v-if="column.sortable" class="sort-mark">{{ sortIndicator(column.key) }}</span>
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
                <td v-if="column.key === 'index'" class="cell--index">
                  {{ startIndex + index + 1 }}
                </td>
                <td v-else-if="column.key === 'actions'" class="actions cell--actions" :style="{ width: column.width, minWidth: column.width, maxWidth: column.width, boxSizing: 'border-box', whiteSpace: 'nowrap' }">
                  <button type="button" class="text-button" @click="handleAction('detail', row)">盘点详情</button>
                  <button type="button" class="text-button" @click="handleAction('photo', row)">盘点人照片</button>
                  <button 
                    type="button"
                    class="text-button" 
                    :class="{ 'text-button--disabled': row.status !== '已完成' }"
                    :title="row.status !== '已完成' ? '暂未生成盘点报告！' : ''"
                    @click="handleAction('report', row)"
                  >盘点报告</button>
                </td>
                <td v-else-if="column.key === 'status'">
                  <span class="chip" :class="row.status === '已完成' ? 'chip--success' : 'chip--warning'">{{ row.status }}</span>
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
