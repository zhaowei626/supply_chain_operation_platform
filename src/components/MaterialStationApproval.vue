<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const approvalTypeOptions = ['材料站注册', '材料站变更', '材料站启用', '材料站停用']
const approvalStatusOptions = ['待审批', '已通过', '已驳回']

const tableColumns = [
  { key: 'selection', label: '', width: '48px', sortable: false },
  { key: 'index', label: '序号', width: '60px', sortable: false },
  { key: 'type', label: '审批类型', width: '130px', sortable: true },
  { key: 'fullName', label: '材料站全称', width: '200px', sortable: true },
  { key: 'code', label: '材料站编码', width: '140px', sortable: true },
  { key: 'project', label: '关联项目', width: '180px', sortable: true },
  { key: 'applicant', label: '申请人', width: '100px', sortable: true },
  { key: 'applicantPhone', label: '申请人联系方式', width: '140px', sortable: true },
  { key: 'companyDept', label: '所属公司/部门', width: '180px', sortable: true },
  { key: 'applyTime', label: '申请时间', width: '160px', sortable: true },
  { key: 'status', label: '审批状态', width: '100px', sortable: true },
  { key: 'actions', label: '操作', width: '140px', sortable: false },
]

const approvalRows = [
  { id: 1, type: '材料站注册', fullName: '沈阳铁西物资供应中心材料站', code: 'XRAA001', project: '沈阳500kV盛京变电站扩建工程', applicant: '刘海', applicantPhone: '138-1234-5678', companyDept: '国网沈阳供电公司 / 物资部', applyTime: '2026-10-10 10:00', status: '待审批' },
  { id: 2, type: '材料站变更', fullName: '大连甘井子物资供应中心材料站', code: 'XRBB002', project: '大连220kV南部电网网架完善工程', applicant: '陈敏', applicantPhone: '139-8765-4321', companyDept: '国网大连供电公司 / 运检部', applyTime: '2026-10-09 15:30', status: '已通过' },
  { id: 3, type: '材料站启用', fullName: '鞍山铁东物资周转站材料站', code: 'XRCC003', project: '鞍山10kV农网改造升级项目', applicant: '周立', applicantPhone: '137-5555-6666', companyDept: '国网鞍山供电公司 / 营销部', applyTime: '2026-10-09 09:15', status: '已驳回' },
  { id: 4, type: '材料站停用', fullName: '抚顺顺城物资储备库材料站', code: 'XRDD004', project: '抚顺66kV城东变电站新建工程', applicant: '韩雪', applicantPhone: '158-9999-8888', companyDept: '国网抚顺供电公司 / 财务部', applyTime: '2026-10-08 14:20', status: '待审批' },
  { id: 5, type: '材料站注册', fullName: '本溪明山材料站材料站', code: 'XREE005', project: '本溪500kV徐家变电站改造工程', applicant: '王强', applicantPhone: '186-1111-2222', companyDept: '国网本溪供电公司 / 发建部', applyTime: '2026-10-08 11:10', status: '已通过' },
  { id: 6, type: '材料站变更', fullName: '锦州凌河物资供应站材料站', code: 'XRFF006', project: '锦州220kV松山输电线路工程', applicant: '李珂', applicantPhone: '189-3333-4444', companyDept: '国网锦州供电公司 / 物资部', applyTime: '2026-10-07 16:45', status: '待审批' },
]

const createEmptyForm = () => ({
  fullName: '',
  type: '',
  status: '',
  project: '',
  applicant: '',
  applyTimeStart: '',
  applyTimeEnd: '',
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())
const currentPage = ref(1)
const pageSize = ref(10)
const sortState = ref({ key: 'applyTime', direction: 'desc' })
const loading = ref(true)
const selectedRows = ref([])

const batchApproveConfirmVisible = ref(false)
const batchApproveMessage = ref('')

let loadingTimer = null

const filteredRows = computed(() => {
  const filters = activeFilters.value
  return approvalRows.filter((row) => {
    const textMatch = (source, query) => !query || String(source).toLowerCase().includes(query.toLowerCase())
    
    let timeMatch = true
    if (filters.applyTimeStart || filters.applyTimeEnd) {
      const rowDate = row.applyTime.split(' ')[0]
      if (filters.applyTimeStart && rowDate < filters.applyTimeStart) timeMatch = false
      if (filters.applyTimeEnd && rowDate > filters.applyTimeEnd) timeMatch = false
    }

    return (
      textMatch(row.fullName, filters.fullName) &&
      textMatch(row.project, filters.project) &&
      textMatch(row.applicant, filters.applicant) &&
      (!filters.type || row.type === filters.type) &&
      (!filters.status || row.status === filters.status) &&
      timeMatch
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

const statusTone = (value) => {
  if (value === '已通过') return 'success'
  if (value === '已驳回') return 'danger'
  return 'neutral'
}

const formatTextTruncate = (text, maxLength = 10) => {
  if (!text) return ''
  if (text.length > maxLength) {
    return text.substring(0, maxLength) + '...'
  }
  return text
}

function handleBatchApprove() {
  if (selectedRows.value.length === 0) {
    alert('请先选择要审批的数据')
    return
  }
  batchApproveMessage.value = `${selectedRows.value.join('、')}，上述材料站的申请确认审批通过吗？`
  batchApproveConfirmVisible.value = true
}

function confirmBatchApprove() {
  alert('批量审批操作成功')
  batchApproveConfirmVisible.value = false
  selectedRows.value = []
}

function handleBatchReject() {
  if (selectedRows.value.length === 0) {
    alert('请先选择要驳回的数据')
    return
  }
  alert('已批量驳回：' + selectedRows.value.join(', '))
  selectedRows.value = []
}

function handleExport() {
  alert('触发了导出操作')
}

function handleAudit(row) {
  alert('审核：' + row.fullName)
}

function handleDetail(row) {
  alert('详情：' + row.fullName)
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
          <span>材料站全称</span>
          <input v-model.trim="form.fullName" type="text" placeholder="请输入材料站全称" />
        </label>
        <label class="field">
          <span>审批类型</span>
          <select v-model="form.type">
            <option value="">全部类型</option>
            <option v-for="option in approvalTypeOptions" :key="option" :value="option">{{ option }}</option>
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
        <label class="field">
          <span>申请人</span>
          <input v-model.trim="form.applicant" type="text" placeholder="请输入申请人" />
        </label>
        <label class="field" style="grid-column: span 12;">
          <span>申请时间范围</span>
          <div style="display: flex; align-items: center; gap: 8px; flex: 1;">
            <input v-model="form.applyTimeStart" type="date" style="flex: 1;" />
            <span style="padding: 0; min-width: auto; text-align: center;">-</span>
            <input v-model="form.applyTimeEnd" type="date" style="flex: 1;" />
          </div>
        </label>
        
        <div class="filter-actions" style="grid-column: span 6;">
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
            批量审批通过
          </button>
          <button class="btn btn--ghost" type="button" @click="handleBatchReject">
            <span class="btn__icon">✗</span>
            批量驳回
          </button>
          <button class="btn btn--ghost" type="button" @click="handleExport">
            <span class="btn__icon">↓</span>
            导出
          </button>
        </div>
      </div>
      <div class="table-wrap">
        <table class="data-table" style="min-width: 1200px;">
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
              <td><span class="chip chip--soft">{{ row.type }}</span></td>
              <td :title="row.fullName">{{ formatTextTruncate(row.fullName, 12) }}</td>
              <td>{{ row.code }}</td>
              <td :title="row.project">{{ formatTextTruncate(row.project, 12) }}</td>
              <td>{{ row.applicant }}</td>
              <td>{{ row.applicantPhone }}</td>
              <td :title="row.companyDept">{{ formatTextTruncate(row.companyDept, 12) }}</td>
              <td>{{ row.applyTime }}</td>
              <td>
                <span class="chip" :class="`chip--${statusTone(row.status)}`">{{ row.status }}</span>
              </td>
              <td class="actions cell--actions">
                <button class="text-button" type="button" v-if="row.status === '待审批'" @click="handleAudit(row)">审核</button>
                <button class="text-button" type="button" @click="handleDetail(row)">详情</button>
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

    <!-- 批量审批通过二次确认框 -->
    <teleport to="body">
      <div v-if="batchApproveConfirmVisible" class="modal-overlay" style="z-index: 1100; display: flex; align-items: center; justify-content: center;" @click.self="batchApproveConfirmVisible = false">
        <div class="modal-dialog" style="width: 480px; max-width: 90vw; border-radius: 12px; overflow: hidden; margin: 0;">
          <div class="modal-header" style="padding: 16px 24px; border-bottom: 1px solid var(--line);">
            <h3 class="modal-title" style="font-size: 16px; margin: 0;">批量审批通过确认</h3>
            <button class="icon-button icon-button--ghost" style="width: 28px; height: 28px; border-radius: 6px;" @click="batchApproveConfirmVisible = false">
              <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body" style="padding: 24px; font-size: 14px; line-height: 1.5;">
            {{ batchApproveMessage }}
          </div>
          <div class="modal-footer" style="padding: 16px 24px; border-top: 1px solid var(--line); display: flex; justify-content: flex-end; gap: 12px; background: #fff;">
            <button class="btn btn--ghost" @click="batchApproveConfirmVisible = false">取消</button>
            <button class="btn btn--primary" @click="confirmBatchApprove">确认</button>
          </div>
        </div>
      </div>
    </teleport>
  </div>
</template>
