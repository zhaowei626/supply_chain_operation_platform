<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const applicationTypeOptions = ['现场耗用', '转材料站', '现场退料', '库存盘点']
const approvalStatusOptions = ['待审批', '已通过', '已驳回']

const tableColumns = [
  { key: 'selection', label: '', width: '48px', sortable: false },
  { key: 'index', label: '序号', width: '60px', sortable: false },
  { key: 'code', label: '申请单号', width: '160px', sortable: true },
  { key: 'type', label: '申请类型', width: '130px', sortable: true },
  { key: 'stationName', label: '材料站全称', width: '180px', sortable: true },
  { key: 'applicant', label: '申请人', width: '100px', sortable: true },
  { key: 'applicantPhone', label: '申请人联系方式', width: '140px', sortable: true },
  { key: 'companyDept', label: '所属公司/部门', width: '160px', sortable: true },
  { key: 'applyTime', label: '申请时间', width: '150px', sortable: true },
  { key: 'status', label: '审批状态', width: '100px', sortable: true },
  { key: 'actions', label: '操作', width: '140px', sortable: false },
]

const approvalRows = [
  { id: 0, code: 'ZCLZ20260928001', type: '转材料站', stationName: '国网沈阳供电公司铁西物资供应中心', project: '沈阳500kV盛京变电站扩建工程', applicant: '刘海', applicantPhone: '13812345678', companyDept: '国网沈阳供电公司 / 物资部', applyTime: '2026-09-28 10:00', status: '待审批' },
  { id: 1, code: 'XCTL20260927001', type: '现场退料', stationName: '国网大连供电公司甘井子物资供应中心', project: '大连220kV南部电网网架完善工程', applicant: '陈敏', applicantPhone: '13987654321', companyDept: '国网大连供电公司 / 运检部', applyTime: '2026-09-27 15:30', status: '已通过' },
  { id: 2, code: 'ZCLZ20260927001', type: '转材料站', stationName: '国网鞍山供电公司铁东物资周转站', project: '鞍山10kV农网改造升级项目', applicant: '周立', applicantPhone: '13755556666', companyDept: '国网鞍山供电公司 / 营销部', applyTime: '2026-09-27 09:15', status: '已驳回' },
  { id: 3, code: 'XCHY20260926001', type: '现场耗用', stationName: '国网抚顺供电公司顺城物资储备库', project: '抚顺66kV城东变电站新建工程', applicant: '韩雪', applicantPhone: '15899998888', companyDept: '国网抚顺供电公司 / 财务部', applyTime: '2026-09-26 14:20', status: '待审批' },
  { id: 4, code: 'ZCLZ20260926001', type: '转材料站', stationName: '国网本溪供电公司明山材料站', project: '本溪500kV徐家变电站改造工程', applicant: '王强', applicantPhone: '18611112222', companyDept: '国网本溪供电公司 / 发建部', applyTime: '2026-09-26 11:10', status: '已通过' },
  { id: 5, code: 'XCTL20260925001', type: '现场退料', stationName: '国网锦州供电公司凌河物资供应站', project: '锦州220kV松山输电线路工程', applicant: '李珂', applicantPhone: '18933334444', companyDept: '国网锦州供电公司 / 物资部', applyTime: '2026-09-25 16:45', status: '待审批' },
  { id: 6, code: 'KCPD20260929003', type: '库存盘点', stationName: '国网丹东供电公司物资库', project: '丹东配网改造项目', applicant: '赵强', applicantPhone: '13511112222', companyDept: '国网丹东供电公司 / 物资部', applyTime: '2026-09-29 09:30', status: '已通过' },
  { id: 7, code: 'KCPD20260929004', type: '库存盘点', stationName: '国网营口供电公司物资库', project: '营口配网改造项目', applicant: '孙伟', applicantPhone: '13622223333', companyDept: '国网营口供电公司 / 物资部', applyTime: '2026-09-29 10:15', status: '待审批' },
]

const createEmptyForm = () => ({
  code: '',
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

// Modal State
const detailModalVisible = ref(false)
const activeDetailTab = ref('info')
const currentDetail = ref(null)
const modalMode = ref('detail') // 'detail' | 'approve'

// Approval Actions State
const approveConfirmVisible = ref(false)
const rejectPromptVisible = ref(false)
const rejectReason = ref('')

// Sign Report Modal State
const signReportModalVisible = ref(false)
const leaderSignature = ref({
  photo: '', // Start empty to trigger the required check
  name: '王建国',
  position: '部门经理',
  signTime: ''
})

function handleSignReport() {
  const now = new Date()
  const year = now.getFullYear()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const date = String(now.getDate()).padStart(2, '0')
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  leaderSignature.value.signTime = `${year}-${month}-${date} ${hours}:${minutes}`
  leaderSignature.value.photo = '' // empty for test
  signReportModalVisible.value = true
}

function confirmSignReport() {
  if (!leaderSignature.value.photo) {
    alert('请联系管理员，更新领导签名照片')
    return
  }
  alert('报告已生成并签署')
  signReportModalVisible.value = false
  closeDetail()
}

function handlePass() {
  approveConfirmVisible.value = true
}

function handleReject() {
  rejectReason.value = ''
  rejectPromptVisible.value = true
}

function confirmPass() {
  alert('审批已通过')
  approveConfirmVisible.value = false
  closeDetail()
}

function confirmReject() {
  if (!rejectReason.value.trim()) {
    alert('请输入驳回原因')
    return
  }
  alert('审批已驳回，原因：' + rejectReason.value)
  rejectPromptVisible.value = false
  closeDetail()
}

// Detail Modal Pagination State
const detailCurrentPage = ref(1)
const detailPageSize = ref(5)

const paginatedMaterialList = computed(() => {
  if (!currentDetail.value || !currentDetail.value.materialList) return []
  const start = (detailCurrentPage.value - 1) * detailPageSize.value
  return currentDetail.value.materialList.slice(start, start + detailPageSize.value)
})

const detailPageCount = computed(() => {
  if (!currentDetail.value || !currentDetail.value.materialList) return 1
  return Math.max(1, Math.ceil(currentDetail.value.materialList.length / detailPageSize.value))
})

const detailStartIndex = computed(() => (detailCurrentPage.value - 1) * detailPageSize.value)

function gotoDetailPage(page) {
  detailCurrentPage.value = Math.min(Math.max(1, page), detailPageCount.value)
}

function openDetail(row, mode = 'detail') {
  modalMode.value = mode
    const realMaterials = [
      { desc: '交流悬式瓷绝缘子 U160BLP', unit: '片' },
      { desc: '钢芯铝绞线 JL/G1A-400/35', unit: 't' },
      { desc: '10kV交联聚乙烯绝缘电力电缆 YJV22-8.7/15kV-3×400', unit: 'km' },
      { desc: '油浸式变压器 S11-M-400/10', unit: '台' },
      { desc: '高压交流隔离开关 GW4-126D(W) / 1250', unit: '组' },
      { desc: '复合绝缘子 FXBW4-110/100', unit: '支' },
      { desc: '碳芯铝绞线 JLRX/F1A-300/40', unit: 't' },
      { desc: '热镀锌钢管 Φ114×4.5', unit: 't' },
      { desc: '跌落式熔断器 RW11-12/100A', unit: '组' },
      { desc: '氧化锌避雷器 YH5WS-17/50', unit: '只' },
      { desc: '金具 悬垂线夹 XGU-4', unit: '套' },
      { desc: '预应力混凝土电杆 Φ190×12m', unit: '根' }
    ]

    const generateMockMaterials = (project) => {
      return Array.from({ length: 12 }, (_, i) => {
        const prefix = ['50', '41', '43'][Math.floor(Math.random() * 3)]
        const purchaseOrderNo = prefix + String(Math.floor(Math.random() * 10000000)).padStart(7, '0')
        const materialCode = '50' + String(Math.floor(Math.random() * 1000000)).padStart(6, '0')
        const orderType = Math.random() > 0.5 ? '电商' : '非电商'
        const mat = realMaterials[i % realMaterials.length]
        return {
          id: i + 1,
          purchaseOrderNo,
          lineItem: String((i + 1) * 10),
          materialCode,
          materialDesc: mat.desc,
          quantity: Math.floor(Math.random() * 100) + 1,
          unit: mat.unit,
          project: project,
          orderType
        }
      })
    }

  detailCurrentPage.value = 1

  const realStations = [
    { name: '国网沈阳供电公司铁西物资供应中心', code: 'ST-SY-001', address: '辽宁省沈阳市铁西区北二中路18号' },
    { name: '国网大连供电公司甘井子物资供应中心', code: 'ST-DL-002', address: '辽宁省大连市甘井子区华北路120号' },
    { name: '国网鞍山供电公司铁东物资周转站', code: 'ST-AS-003', address: '辽宁省鞍山市铁东区千山中路33号' },
    { name: '国网抚顺供电公司顺城物资储备库', code: 'ST-FS-004', address: '辽宁省抚顺市顺城区新城路中段50号' },
    { name: '国网本溪供电公司明山材料站', code: 'ST-BX-005', address: '辽宁省本溪市明山区解放北路88号' },
    { name: '国网锦州供电公司凌河物资供应站', code: 'ST-JZ-006', address: '辽宁省锦州市凌河区解放路五段9号' }
  ]
  const station = realStations[row.id % realStations.length || 0]
  const targetStation = realStations[(row.id + 1) % realStations.length || 0]

  const generateTimeline = (row) => {
    const timeline = []
    let currentId = 1
    const time1 = row.applyTime + ' 09:00'
    const time2 = row.applyTime + ' 14:30'
    const time3 = row.applyTime + ' 16:00'
    const time4 = row.applyTime + ' 17:30'

    // 针对特定单据定制轨迹
    if (row.code === 'XCHY20260926001') {
      timeline.push({ id: currentId++, nodeName: '提交申请', operator: row.applicant, time: time1, result: '提交成功', comment: '' })
      timeline.push({ id: currentId++, nodeName: '领导审批', operator: '王建国', time: time2, result: '已驳回', comment: '材料清单填写有误，请核对后重新提交。' })
      timeline.push({ id: currentId++, nodeName: '再次申请', operator: row.applicant, time: time3, result: '提交成功', comment: '已修改材料清单并重新提交' })
      timeline.push({ id: currentId++, nodeName: '领导审批', operator: '王建国', time: time4, result: '待审批', comment: '' })
      return timeline.reverse()
    }

    // 模拟部分数据有驳回后再次申请的历史循环
    const hasRejectionHistory = row.id % 2 === 0

    if (hasRejectionHistory) {
      timeline.push({ id: currentId++, nodeName: '提交申请', operator: row.applicant, time: time1, result: '提交成功', comment: '' })
      timeline.push({ id: currentId++, nodeName: '领导审批', operator: '王建国', time: time2, result: '已驳回', comment: '材料清单填写有误，请核对后重新提交。' })
      timeline.push({ id: currentId++, nodeName: '再次申请', operator: row.applicant, time: time3, result: '提交成功', comment: '已修改材料清单并重新提交' })
      timeline.push({
        id: currentId++,
        nodeName: '领导审批',
        operator: '王建国',
        time: time4,
        result: row.status,
        comment: row.status === '已驳回' ? '数量依然超标，不予通过' : (row.status === '待审批' ? '' : '情况属实，同意领用')
      })
    } else {
      timeline.push({ id: currentId++, nodeName: '提交申请', operator: row.applicant, time: time1, result: '提交成功', comment: '' })
      timeline.push({
        id: currentId++,
        nodeName: '领导审批',
        operator: '王建国',
        time: time2,
        result: row.status,
        comment: row.status === '已驳回' ? '库存不足，请暂缓领用' : (row.status === '待审批' ? '' : '同意')
      })
    }
    
    // 按时间倒序排列（最新的节点在最上面）
    return timeline.reverse()
  }

  const returnReasons = ['施工方案调整', '物料型号规格不匹配', '物资外观瑕疵', '领用计划与实际耗用偏差', '其他']
  const returnReason = returnReasons[Math.floor(Math.random() * returnReasons.length)]
  const relatedApplyCode = 'XCHY' + String(Math.floor(Math.random() * 10000000)).padStart(7, '0') + '001'

  const yearMonth = row.applyTime ? row.applyTime.substring(0, 4) + '年' + row.applyTime.substring(5, 7) + '月' : '2026年09月'
  const inventoryRecordName = `${station.name}-${yearMonth}-第${Math.floor(Math.random() * 5) + 1}次盘点`

  currentDetail.value = {
    ...row,
    description: '现场施工急需，用于一期工程基础建设，请尽快审批发放。',
    photos: ['photo1', 'photo2'],
    stationName: station.name,
    stationCode: station.code,
    stationAddress: station.address,
    targetStationName: targetStation.name,
    targetStationCode: targetStation.code,
    targetStationAddress: targetStation.address,
    relatedApplyCode: relatedApplyCode,
    returnReason: returnReason,
    returnDescription: '施工结束后剩余物料，现申请退库处理。',
    inventoryRecordName: inventoryRecordName,
    signName: '王建国',
    signPosition: '部门经理',
    signTime: '2026-09-30 15:30',
    materialList: generateMockMaterials(row.project),
    timeline: generateTimeline(row)
  }
  activeDetailTab.value = mode === 'process' ? 'approval' : 'info'
  detailModalVisible.value = true
}

function closeDetail() {
  detailModalVisible.value = false
  currentDetail.value = null
}

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

const batchApproveConfirmVisible = ref(false)
const batchApproveMessage = ref('')

const batchRejectPromptVisible = ref(false)
const batchRejectMessage = ref('')
const batchRejectReason = ref('')

function handleBatchApprove() {
  if (selectedRows.value.length === 0) {
    alert('请先选择要审批的数据')
    return
  }

  const selectedTypes = new Set()
  selectedRows.value.forEach(code => {
    const row = approvalRows.find(r => r.code === code)
    if (row) {
      selectedTypes.add(row.type)
    }
  })

  if (selectedTypes.size > 1) {
    alert('所选记录包含不同的申请类型，请选择相同申请类型的单据进行批量审批！')
    return
  }

  batchApproveMessage.value = `${selectedRows.value.join('、')}，上述申请单确认审批通过吗？`
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
  batchRejectMessage.value = `已选定${selectedRows.value.join('、')}申请单`
  batchRejectReason.value = ''
  batchRejectPromptVisible.value = true
}

function confirmBatchReject() {
  if (!batchRejectReason.value.trim()) {
    alert('请输入驳回原因')
    return
  }
  alert(`批量驳回操作成功，原因：${batchRejectReason.value}`)
  batchRejectPromptVisible.value = false
  selectedRows.value = []
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
    
    let timeMatch = true
    if (filters.applyTimeStart || filters.applyTimeEnd) {
      const rowDate = row.applyTime.split(' ')[0]
      if (filters.applyTimeStart && rowDate < filters.applyTimeStart) timeMatch = false
      if (filters.applyTimeEnd && rowDate > filters.applyTimeEnd) timeMatch = false
    }

    return (
      textMatch(row.code, filters.code) &&
      textMatch(row.stationName, filters.project) &&
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
            <option value="">请选择申请类型</option>
            <option v-for="option in applicationTypeOptions" :key="option" :value="option">{{ option }}</option>
          </select>
        </label>
        <label class="field">
          <span>审批状态</span>
          <select v-model="form.status">
            <option value="">请选择审批状态</option>
            <option v-for="option in approvalStatusOptions" :key="option" :value="option">{{ option }}</option>
          </select>
        </label>
        <label class="field">
          <span>材料站全称</span>
          <input v-model.trim="form.project" type="text" placeholder="请输入材料站全称" />
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
              <td :title="row.stationName">{{ formatTextTruncate(row.stationName) }}</td>
              <td>{{ row.applicant }}</td>
              <td>{{ row.applicantPhone }}</td>
              <td :title="row.companyDept">{{ formatTextTruncate(row.companyDept) }}</td>
              <td>{{ row.applyTime }}</td>
              <td>
                <span class="chip" :class="`chip--${statusTone(row.status)}`">{{ row.status }}</span>
              </td>
              <td class="actions cell--actions">
                <button class="text-button" type="button" v-if="row.status === '待审批'" @click="openDetail(row, 'approve')">审核</button>
                <button class="text-button" type="button" @click="openDetail(row, 'detail')">详情</button>
                <button class="text-button" type="button" @click="openDetail(row, 'process')">流程</button>
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
</div>
    
    <!-- Detail Modal -->
    <teleport to="body">
      <div v-if="detailModalVisible" class="modal-overlay" @click.self="closeDetail">
        <div class="modal-dialog">
          <div class="modal-header">
            <h2 class="modal-title" v-if="modalMode === 'process'">
              查看流程跟踪
            </h2>
            <h2 class="modal-title" v-else>
              业务审批详情 - 
              <span style="color: #1e3a8a;">{{ currentDetail?.type }}</span> - 
              <span style="color: #9ca3af;">{{ currentDetail?.code }}</span>
            </h2>
            <button class="icon-button icon-button--ghost" style="width: 32px; height: 32px; border-radius: 8px;" @click="closeDetail">
              <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <!-- Tab bar is now completely removed since detail, process, and approve modes all only show single content types -->
          <div class="modal-body">
            <!-- Info Content (Visible in detail mode and approve mode) -->
            <div v-if="modalMode === 'approve' || modalMode === 'detail'" class="modal-content">
              <div class="detail-card">
                <div class="detail-grid">
                  <div class="detail-field"><span class="detail-field__label">{{ currentDetail.type === '库存盘点' ? '盘点人' : '申请人' }}</span><span class="detail-field__value">{{ currentDetail.applicant }}</span></div>
                  <div class="detail-field"><span class="detail-field__label">{{ currentDetail.type === '库存盘点' ? '盘点人联系方式' : '申请人联系方式' }}</span><span class="detail-field__value">{{ currentDetail.applicantPhone }}</span></div>
                  <div class="detail-field"><span class="detail-field__label">所属公司/部门</span><span class="detail-field__value">{{ currentDetail.companyDept }}</span></div>
                  <div class="detail-field"><span class="detail-field__label">{{ currentDetail.type === '库存盘点' ? '盘点时间' : '申请时间' }}</span><span class="detail-field__value">{{ currentDetail.applyTime }}</span></div>
                  <div class="detail-field"><span class="detail-field__label">材料站全称</span><span class="detail-field__value">{{ currentDetail.stationName }}</span></div>
                  <div class="detail-field"><span class="detail-field__label">材料站编码</span><span class="detail-field__value">{{ currentDetail.stationCode }}</span></div>
                  <div class="detail-field" style="grid-column: span 2;"><span class="detail-field__label">材料站地址</span><span class="detail-field__value">{{ currentDetail.stationAddress }}</span></div>
                  
                  <template v-if="currentDetail.type === '库存盘点'">
                    <div class="detail-field" style="grid-column: span 2;"><span class="detail-field__label">盘点记录名称</span><span class="detail-field__value">{{ currentDetail.inventoryRecordName }}</span></div>
                    <div class="detail-field" style="grid-column: span 2;">
                      <span class="detail-field__label">盘点人照片</span>
                      <div style="display: flex; gap: 12px; margin-top: 4px;">
                         <div v-for="(photo, i) in currentDetail.photos" :key="i" style="width: 80px; height: 80px; background: #f1f5f9; border-radius: 8px; border: 1px dashed var(--line); display: flex; align-items: center; justify-content: center; color: var(--muted); font-size: 12px;">照片 {{i+1}}</div>
                      </div>
                    </div>
                    <template v-if="currentDetail.status === '已通过'">
                      <div class="detail-field" style="grid-column: span 1;"><span class="detail-field__label">签署人姓名</span><span class="detail-field__value">{{ currentDetail.signName }}</span></div>
                      <div class="detail-field" style="grid-column: span 1;"><span class="detail-field__label">签署人岗位</span><span class="detail-field__value">{{ currentDetail.signPosition }}</span></div>
                      <div class="detail-field" style="grid-column: span 1;"><span class="detail-field__label">签署时间</span><span class="detail-field__value">{{ currentDetail.signTime }}</span></div>
                      <div class="detail-field" style="grid-column: span 1;">
                        <span class="detail-field__label">签署人签名照片</span>
                        <div style="width: 60px; height: 30px; background: #f8fafc; border: 1px dashed var(--line); border-radius: 4px; display: flex; align-items: center; justify-content: center; color: var(--primary); font-size: 12px; margin-top: 4px;">已签名</div>
                      </div>
                    </template>
                  </template>
                  <template v-else-if="currentDetail.type !== '现场退料'">
                    <template v-if="currentDetail.type === '转材料站'">
                      <div class="detail-field" style="grid-column: span 1;"><span class="detail-field__label">目标材料站全称</span><span class="detail-field__value">{{ currentDetail.targetStationName }}</span></div>
                      <div class="detail-field" style="grid-column: span 1;"><span class="detail-field__label">目标材料站编码</span><span class="detail-field__value">{{ currentDetail.targetStationCode }}</span></div>
                      <div class="detail-field" style="grid-column: span 2;"><span class="detail-field__label">目标材料站地址</span><span class="detail-field__value">{{ currentDetail.targetStationAddress }}</span></div>
                    </template>
                    <div class="detail-field" style="grid-column: span 2;"><span class="detail-field__label">领用说明</span><span class="detail-field__value">{{ currentDetail.description }}</span></div>
                    <div class="detail-field" style="grid-column: span 2;">
                      <span class="detail-field__label">领用照片</span>
                      <div style="display: flex; gap: 12px; margin-top: 4px;">
                         <div v-for="(photo, i) in currentDetail.photos" :key="i" style="width: 80px; height: 80px; background: #f1f5f9; border-radius: 8px; border: 1px dashed var(--line); display: flex; align-items: center; justify-content: center; color: var(--muted); font-size: 12px;">照片 {{i+1}}</div>
                      </div>
                    </div>
                  </template>

                  <template v-if="currentDetail.type === '现场退料'">
                       <div class="detail-field" style="grid-column: span 1;">
                         <span class="detail-field__label">关联领用单号</span>
                         <a href="#" class="detail-field__value" style="color: var(--primary); text-decoration: none;" @click.prevent="alert('点击了关联领用单号: ' + currentDetail.relatedApplyCode)">{{ currentDetail.relatedApplyCode }}</a>
                       </div>
                       <div class="detail-field" style="grid-column: span 1;"><span class="detail-field__label">退料原因</span><span class="detail-field__value">{{ currentDetail.returnReason }}</span></div>
                      <div class="detail-field" style="grid-column: span 2;"><span class="detail-field__label">退料说明</span><span class="detail-field__value">{{ currentDetail.returnDescription }}</span></div>
                    </template>
                </div>
              </div>
              
              <div class="detail-card">
                <h3 class="detail-card__title">物资清单</h3>
                <div class="table-wrap" style="border: 1px solid var(--line); border-radius: 8px; border-top: 1px solid var(--line); overflow-x: hidden;">
                  <table class="data-table" style="min-width: 100%; table-layout: fixed;">
                    <thead>
                      <tr>
                        <th style="width: 50px; z-index: 0;">序号</th>
                        <th style="width: 120px; z-index: 0;">采购订单编号</th>
                        <th style="width: 70px; z-index: 0;">行项目</th>
                        <th style="width: 90px; z-index: 0;">物料编码</th>
                        <th style="width: 200px; z-index: 0;">物料描述</th>
                        <th style="width: 80px; z-index: 0;">{{ currentDetail.type === '库存盘点' ? '在库数量' : (currentDetail.type === '现场退料' ? '退料数量' : '领用数量') }}</th>
                        <th style="width: 70px; z-index: 0;">计量单位</th>
                        <th style="width: 150px; z-index: 0;">所属项目</th>
                        <th style="width: 80px; z-index: 0;">{{ currentDetail.type === '库存盘点' ? '实物照片' : '订单类型' }}</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(mat, i) in paginatedMaterialList" :key="mat.id">
                        <td class="cell--index">{{ detailStartIndex + i + 1 }}</td>
                        <td>{{ mat.purchaseOrderNo }}</td>
                        <td>{{ mat.lineItem }}</td>
                        <td>{{ mat.materialCode }}</td>
                        <td :title="mat.materialDesc">{{ formatTextTruncate(mat.materialDesc, 10) }}</td>
                        <td>{{ mat.quantity }}</td>
                        <td>{{ mat.unit }}</td>
                        <td :title="mat.project"><span class="chip chip--soft" style="max-width: 100%; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; display: block;">{{ formatTextTruncate(mat.project, 10) }}</span></td>
                        <td>
                          <a href="#" v-if="currentDetail.type === '库存盘点'" style="color: var(--primary); text-decoration: none;" @click.prevent>查看</a>
                          <template v-else>{{ mat.orderType }}</template>
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>
                <div class="table-footer" style="padding-top: 12px; border-top: none;">
                  <div class="page-size">
                    <span style="margin-right: 8px;">共 {{ currentDetail.materialList.length }} 条</span>
                    每页
                    <select v-model.number="detailPageSize" @change="gotoDetailPage(1)">
                      <option :value="5">5 条</option>
                      <option :value="10">10 条</option>
                      <option :value="20">20 条</option>
                    </select>
                  </div>
                  <div class="pagination">
                    <button class="page-btn" type="button" :disabled="detailCurrentPage === 1" @click="gotoDetailPage(detailCurrentPage - 1)">‹</button>
                    <button
                      v-for="page in detailPageCount"
                      :key="page"
                      class="page-btn"
                      :class="{ 'page-btn--active': detailCurrentPage === page }"
                      type="button"
                      @click="gotoDetailPage(page)"
                    >
                      {{ page }}
                    </button>
                    <button class="page-btn" type="button" :disabled="detailCurrentPage === detailPageCount" @click="gotoDetailPage(detailCurrentPage + 1)">›</button>
                  </div>
                </div>
              </div>
            </div>

            <!-- Approval Content (Visible only in process mode) -->
            <div v-if="modalMode === 'process'" class="modal-content">
              <div class="detail-card">
                <h3 class="detail-card__title">审核轨迹</h3>
                <div class="timeline">
                  <div class="timeline-item" v-for="item in currentDetail.timeline" :key="item.id">
                    <div class="timeline-item__tail"></div>
                    <div class="timeline-item__node"></div>
                    <div class="timeline-item__content">
                      <div class="timeline-item__header">
                        <strong>{{ item.nodeName }}</strong>
                        <span class="timeline-item__time">{{ item.time }}</span>
                      </div>
                      <div class="timeline-item__body">
                        <p><strong>操作人员：</strong>{{ item.operator }}</p>
                        <p><strong>审核结果：</strong><span :class="'chip chip--' + statusTone(item.result)" style="height: 22px; min-height: 22px; font-size: 11px; line-height: 20px; padding: 0 8px;">{{ item.result }}</span></p>
                        <p v-if="item.comment"><strong>审核意见：</strong>{{ item.comment }}</p>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          
          <div class="modal-footer" v-if="modalMode === 'approve'" style="padding: 16px 24px; border-top: 1px solid var(--line); display: flex; justify-content: flex-end; gap: 12px; background: #fff; border-bottom-left-radius: 12px; border-bottom-right-radius: 12px;">
            <template v-if="currentDetail.type === '库存盘点'">
              <button class="btn btn--primary" @click="handleSignReport">签署并生成报告</button>
            </template>
            <template v-else>
              <button class="btn btn--ghost" style="color: #ff4d4f; border-color: #ff4d4f; background: #fff;" @click="handleReject">驳回</button>
              <button class="btn btn--primary" @click="handlePass">审批通过</button>
            </template>
          </div>
        </div>
      </div>

      <!-- 批量审批通过二次确认框 -->
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

      <!-- 批量驳回原因输入框 -->
      <div v-if="batchRejectPromptVisible" class="modal-overlay" style="z-index: 1100; display: flex; align-items: center; justify-content: center;" @click.self="batchRejectPromptVisible = false">
        <div class="modal-dialog" style="width: 520px; max-width: 90vw; border-radius: 12px; overflow: hidden; margin: 0;">
          <div class="modal-header" style="padding: 16px 24px; border-bottom: 1px solid var(--line);">
            <h3 class="modal-title" style="font-size: 16px; margin: 0;">批量驳回申请</h3>
            <button class="icon-button icon-button--ghost" style="width: 28px; height: 28px; border-radius: 6px;" @click="batchRejectPromptVisible = false">
              <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body" style="padding: 24px;">
            <div style="margin-bottom: 16px; font-size: 14px; color: var(--text-secondary); background: #f8fafc; padding: 12px; border-radius: 6px; border: 1px solid var(--line); word-break: break-all; line-height: 1.5;">
              {{ batchRejectMessage }}
            </div>
            <div style="margin-bottom: 8px; font-size: 14px;"><span style="color: #ff4d4f; margin-right: 4px;">*</span>驳回原因</div>
            <textarea v-model="batchRejectReason" rows="4" placeholder="请输入驳回原因..." style="width: 100%; padding: 12px; border: 1px solid var(--line); border-radius: 6px; resize: vertical; font-family: inherit; font-size: 14px; outline: none; transition: border-color 0.2s;" onfocus="this.style.borderColor='var(--primary)'" onblur="this.style.borderColor='var(--line)'"></textarea>
          </div>
          <div class="modal-footer" style="padding: 16px 24px; border-top: 1px solid var(--line); display: flex; justify-content: flex-end; gap: 12px; background: #fff;">
            <button class="btn btn--ghost" @click="batchRejectPromptVisible = false">取消</button>
            <button class="btn btn--primary" style="background: #ff4d4f; border-color: #ff4d4f;" @click="confirmBatchReject">确认驳回</button>
          </div>
        </div>
      </div>

      <!-- 二次确认框 (通过) -->
      <div v-if="approveConfirmVisible" class="modal-overlay" style="z-index: 1100; display: flex; align-items: center; justify-content: center;" @click.self="approveConfirmVisible = false">
        <div class="modal-dialog" style="width: 400px; max-width: 90vw; border-radius: 12px; overflow: hidden; margin: 0;">
          <div class="modal-header" style="padding: 16px 24px; border-bottom: 1px solid var(--line);">
            <h3 class="modal-title" style="font-size: 16px; margin: 0;">审批通过确认</h3>
            <button class="icon-button icon-button--ghost" style="width: 28px; height: 28px; border-radius: 6px;" @click="approveConfirmVisible = false">
              <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body" style="padding: 24px; font-size: 14px;">
            本申请单确认审批通过吗？
          </div>
          <div class="modal-footer" style="padding: 16px 24px; border-top: 1px solid var(--line); display: flex; justify-content: flex-end; gap: 12px; background: #fff;">
            <button class="btn btn--ghost" @click="approveConfirmVisible = false">取消</button>
            <button class="btn btn--primary" @click="confirmPass">确认</button>
          </div>
        </div>
      </div>

      <!-- 驳回原因输入框 -->
      <div v-if="rejectPromptVisible" class="modal-overlay" style="z-index: 1100; display: flex; align-items: center; justify-content: center;" @click.self="rejectPromptVisible = false">
        <div class="modal-dialog" style="width: 480px; max-width: 90vw; border-radius: 12px; overflow: hidden; margin: 0;">
          <div class="modal-header" style="padding: 16px 24px; border-bottom: 1px solid var(--line);">
            <h3 class="modal-title" style="font-size: 16px; margin: 0;">驳回申请</h3>
            <button class="icon-button icon-button--ghost" style="width: 28px; height: 28px; border-radius: 6px;" @click="rejectPromptVisible = false">
              <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body" style="padding: 24px;">
            <div style="margin-bottom: 8px; font-size: 14px;"><span style="color: #ff4d4f; margin-right: 4px;">*</span>驳回原因</div>
            <textarea v-model="rejectReason" rows="4" placeholder="请输入驳回原因..." style="width: 100%; padding: 12px; border: 1px solid var(--line); border-radius: 6px; resize: vertical; font-family: inherit; font-size: 14px; outline: none; transition: border-color 0.2s;" onfocus="this.style.borderColor='var(--primary)'" onblur="this.style.borderColor='var(--line)'"></textarea>
          </div>
          <div class="modal-footer" style="padding: 16px 24px; border-top: 1px solid var(--line); display: flex; justify-content: flex-end; gap: 12px; background: #fff;">
            <button class="btn btn--ghost" @click="rejectPromptVisible = false">取消</button>
            <button class="btn btn--primary" style="background: #ff4d4f; border-color: #ff4d4f;" @click="confirmReject">确认驳回</button>
          </div>
        </div>
      </div>

      <!-- 签署并生成报告弹窗 -->
      <div v-if="signReportModalVisible" class="modal-overlay" style="z-index: 1100; display: flex; align-items: center; justify-content: center;" @click.self="signReportModalVisible = false">
        <div class="modal-dialog" style="width: 400px; max-width: 90vw; border-radius: 12px; overflow: hidden; margin: 0; background: #fff;">
          <div class="modal-header" style="padding: 16px 24px; border-bottom: 1px solid var(--line); display: flex; justify-content: space-between; align-items: center;">
            <h3 class="modal-title" style="font-size: 16px; margin: 0;">签署并生成报告</h3>
            <button class="icon-button icon-button--ghost" style="width: 28px; height: 28px; border-radius: 6px;" @click="signReportModalVisible = false">
              <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
            </button>
          </div>
          <div class="modal-body" style="padding: 24px;">
            <div style="display: flex; flex-direction: column; gap: 16px;">
              <div style="display: flex; justify-content: space-between; align-items: center;">
                <span style="color: var(--muted); font-size: 14px;">姓名</span>
                <span style="font-size: 14px;">{{ leaderSignature.name }}</span>
              </div>
              <div style="display: flex; justify-content: space-between; align-items: center;">
                <span style="color: var(--muted); font-size: 14px;">岗位</span>
                <span style="font-size: 14px;">{{ leaderSignature.position }}</span>
              </div>
              <div style="display: flex; justify-content: space-between; align-items: center;">
                <span style="color: var(--muted); font-size: 14px;">签署时间</span>
                <span style="font-size: 14px;">{{ leaderSignature.signTime }}</span>
              </div>
              <div style="margin-top: 8px;">
                <div style="color: var(--muted); font-size: 14px; margin-bottom: 8px;"><span style="color: #ff4d4f; margin-right: 4px;">*</span>领导手写签名</div>
                <div style="width: 100%; height: 120px; background: #f8fafc; border: 1px dashed var(--line); border-radius: 8px; display: flex; align-items: center; justify-content: center; overflow: hidden;">
                  <img v-if="leaderSignature.photo" :src="leaderSignature.photo" style="max-width: 100%; max-height: 100%; object-fit: contain;" />
                  <div v-else style="display: flex; flex-direction: column; align-items: center; gap: 8px; color: var(--muted);">
                    <svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="none"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline></svg>
                    <span style="font-size: 12px;">空暂未图</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer" style="padding: 16px 24px; border-top: 1px solid var(--line); display: flex; justify-content: flex-end; gap: 12px; background: #fff;">
            <button class="btn btn--ghost" @click="signReportModalVisible = false">取消</button>
            <button class="btn btn--primary" @click="confirmSignReport">确定</button>
          </div>
        </div>
      </div>
    </teleport>
</template>
