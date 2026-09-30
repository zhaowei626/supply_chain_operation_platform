<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const applicationTypeOptions = ['现场耗用', '转材料站', '现场退料']
const approvalStatusOptions = ['待审批', '已通过', '已驳回']

const tableColumns = [
  { key: 'selection', label: '', width: '48px', sortable: false },
  { key: 'index', label: '序号', width: '60px', sortable: false },
  { key: 'code', label: '申请单号', width: '160px', sortable: true },
  { key: 'type', label: '申请类型', width: '130px', sortable: true },
  { key: 'project', label: '关联项目', width: '180px', sortable: true },
  { key: 'applicant', label: '申请人', width: '100px', sortable: true },
  { key: 'companyDept', label: '所属公司/部门', width: '160px', sortable: true },
  { key: 'applyTime', label: '申请时间', width: '150px', sortable: true },
  { key: 'status', label: '审批状态', width: '100px', sortable: true },
  { key: 'actions', label: '操作', width: '140px', sortable: false },
]

const approvalRows = [
  { code: 'APP-20260928-001', type: '转材料站', project: 'G312国道苏州段改扩建工程一期项目', applicant: '刘海', companyDept: '华东建工集团 / 物资设备部', applyTime: '2026-09-28 10:00', status: '待审批' },
  { code: 'APP-20260928-002', type: '现场退料', project: '滨江新城商业综合体二期建设项目', applicant: '陈敏', companyDept: '中铁建设二局 / 项目管理部', applyTime: '2026-09-27 15:30', status: '已通过' },
  { code: 'APP-20260927-003', type: '转材料站', project: '南环高速公路路面综合提升工程', applicant: '周立', companyDept: '南方路桥工程 / 综合管理部', applyTime: '2026-09-27 09:15', status: '已驳回' },
  { code: 'APP-20260927-004', type: '现场耗用', project: '城东保障房项目', applicant: '韩雪', companyDept: '北方城建集团 / 成本合约部', applyTime: '2026-09-26 14:20', status: '待审批' },
  { code: 'APP-20260926-005', type: '转材料站', project: 'G312改扩建项目', applicant: '王强', companyDept: '华东建工 / 综合部', applyTime: '2026-09-26 11:10', status: '已通过' },
  { code: 'APP-20260925-006', type: '现场退料', project: '南环高速提升工程', applicant: '李珂', companyDept: '南方路桥工程 / 项目管理部', applyTime: '2026-09-25 16:45', status: '待审批' },
]

const createEmptyForm = () => ({
  code: '',
  type: '',
  status: '',
  project: '',
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())
const currentPage = ref(1)
const pageSize = ref(8)
const sortState = ref({ key: 'applyTime', direction: 'desc' })
const loading = ref(true)
const selectedRows = ref([])

let loadingTimer = null

const selectableRows = computed(() => {
  return paginatedRows.value.filter(row => row.status === '待审批')
})

const isAllSelected = computed(() => {
  return selectableRows.value.length > 0 && selectedRows.value.length === selectableRows.value.length
})

const isIndeterminate = computed(() => {
  return selectedRows.value.length > 0 && selectedRows.value.length < selectableRows.value.length
})

function toggleAllSelection(event) {
  if (event.target.checked) {
    selectedRows.value = selectableRows.value.map(row => row.code)
  } else {
    selectedRows.value = []
  }
}

function handleBatchApprove() {
  if (selectedRows.value.length === 0) {
    alert('请先选择要审批的数据')
    return
  }
  alert(`批量审批了 ${selectedRows.value.length} 条数据: ${selectedRows.value.join(', ')}`)
}

function handleExport() {
  alert('触发了导出操作')
}

const formatTextTruncate = (text, maxLength = 10) => {
  if (!text) return ''
  if (text.length > maxLength) {
    return text.substring(0, maxLength) + '...'
  }
  return text
}

const statusTone = (value) => {
  if (value === '已通过') return 'success'
  if (value === '已驳回') return 'danger'
  return 'neutral'
}

const filteredRows = computed(() => {
  const filters = activeFilters.value
  return approvalRows.filter((row) => {
    const textMatch = (source, query) => !query || String(source).toLowerCase().includes(query.toLowerCase())
    return (
      textMatch(row.code, filters.code) &&
      textMatch(row.project, filters.project) &&
      (!filters.type || row.type === filters.type) &&
      (!filters.status || row.status === filters.status)
    )
  })
})

const sortedRows = computed(() => {
  const rows = [...filteredRows.value]
  const { key, direction } = sortState.value
  rows.sort((left, right) => {
    let leftValue = left[key]
    let rightValue = right[key]
    if (key === 'status') {
      const order = { '待审批': 0, '已通过': 1, '已驳回': 2 }
      leftValue = order[left.status]
      rightValue = order[right.status]
    }
    if (leftValue < rightValue) return direction === 'asc' ? -1 : 1
    if (leftValue > rightValue) return direction === 'asc' ? 1 : -1
    return 0
  })
  return rows
})

const pageCount = computed(() => Math.max(1, Math.ceil(filteredRows.value.length / pageSize.value)))
const paginatedRows = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  return sortedRows.value.slice(start, start + pageSize.value)
})
const startIndex = computed(() => (currentPage.value - 1) * pageSize.value)

function toggleSort(key) {
  if (sortState.value.key === key) {
    sortState.value = { key, direction: sortState.value.direction === 'asc' ? 'desc' : 'asc' }
  } else {
    sortState.value = { key, direction: 'asc' }
  }
}

function sortIndicator(key) {
  if (sortState.value.key !== key) return '↕'
  return sortState.value.direction === 'asc' ? '↑' : '↓'
}

function applyFilters() {
  activeFilters.value = { ...form }
  currentPage.value = 1
  startLoading()
}

function resetFilters() {
  Object.assign(form, createEmptyForm())
  activeFilters.value = createEmptyForm()
  currentPage.value = 1
  sortState.value = { key: 'applyTime', direction: 'desc' }
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
          <span>申请单号</span>
          <input v-model.trim="form.code" type="text" placeholder="请输入申请单号" />
        </label>
        <label class="field">
          <span>申请类型</span>
          <select v-model="form.type">
            <option value="">全部类型</option>
            <option v-for="option in applicationTypeOptions" :key="option" :value="option">{{ option }}</option>
          </select>
        </label>
        <label class="field">
          <span>审批状态</span>
          <select v-model="form.status">
            <option value="">全部状态</option>
            <option v-for="option in approvalStatusOptions" :key="option" :value="option">{{ option }}</option>
          </select>
        </label>
        <label class="field">
          <span>关联项目</span>
          <input v-model.trim="form.project" type="text" placeholder="请输入关联项目" />
        </label>
        
        <div class="filter-actions" style="grid-column: span 24; justify-content: flex-end;">
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
      <div class="results-bar">
        <div class="toolbar">
          <button class="btn btn--ghost" type="button" @click="handleBatchApprove">
            <span class="btn__icon">✓</span>
            批量审核
          </button>
          <button class="btn btn--ghost" type="button" @click="handleExport">
            <span class="btn__icon">↓</span>
            导出
          </button>
        </div>
      </div>
      <div class="table-wrap">
        <table class="data-table" style="min-width: 1000px;">
          <thead>
            <tr>
              <th
                v-for="column in tableColumns"
                :key="column.key"
                :style="{ minWidth: column.width, width: column.width }"
                :class="[
                  column.sortable && 'is-sortable',
                  sortState.key === column.key && 'is-sorted',
                  column.key === 'actions' && 'cell--actions',
                ]"
                @click="column.sortable && toggleSort(column.key)"
              >
                <template v-if="column.key === 'selection'">
                  <input type="checkbox" class="table-checkbox" :checked="isAllSelected" :indeterminate="isIndeterminate" @change="toggleAllSelection" />
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
              <td v-for="column in tableColumns" :key="column.key" :class="[column.key === 'actions' && 'cell--actions']">
                <span class="skeleton-line"></span>
              </td>
            </tr>
          </tbody>
          <tbody v-else-if="paginatedRows.length">
            <tr v-for="(row, index) in paginatedRows" :key="row.code">
              <td>
                <input v-if="row.status === '待审批'" type="checkbox" class="table-checkbox" :value="row.code" v-model="selectedRows" />
              </td>
              <td class="cell--index">{{ startIndex + index + 1 }}</td>
              <td>{{ row.code }}</td>
              <td><span class="chip chip--soft">{{ row.type }}</span></td>
              <td :title="row.project">{{ formatTextTruncate(row.project) }}</td>
              <td>{{ row.applicant }}</td>
              <td :title="row.companyDept">{{ formatTextTruncate(row.companyDept) }}</td>
              <td>{{ row.applyTime }}</td>
              <td>
                <span class="chip" :class="`chip--${statusTone(row.status)}`">{{ row.status }}</span>
              </td>
              <td class="actions cell--actions">
                <button class="text-button" type="button" v-if="row.status === '待审批'">审核</button>
                <button class="text-button" type="button">详情</button>
              </td>
            </tr>
          </tbody>
          <tbody v-else>
            <tr>
              <td :colspan="tableColumns.length">
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
          每页
          <select v-model.number="pageSize" @change="gotoPage(1)">
            <option :value="8">8 条</option>
            <option :value="10">10 条</option>
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
