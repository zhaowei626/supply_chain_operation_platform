<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'
import BusinessApproval from './components/BusinessApproval.vue'
import MaterialStationApproval from './components/MaterialStationApproval.vue'

const menuGroups = reactive([
  {
    title: '材料站管理',
    icon: '◈',
    expanded: true,
    items: [
      { id: 'material-ledger', label: '材料站台账', icon: '◦', active: true },
      { id: 'material-approval', label: '材料站审批', icon: '◦', active: false },
    ],
  },
  {
    title: '业务管理',
    icon: '✦',
    expanded: true,
    items: [
      { id: 'business-approval', label: '业务审批', icon: '◦', active: false },
      { id: 'income-inbound', label: '收入入站明细', icon: '◦', active: false },
      { id: 'site-consumption', label: '现场耗用明细', icon: '◦', active: false },
      { id: 'transfer-material', label: '转材料站明细', icon: '◦', active: false },
      { id: 'site-return', label: '现场退料明细', icon: '◦', active: false },
    ],
  },
  {
    title: '库存管理',
    icon: '▣',
    expanded: true,
    items: [
      { id: 'inventory-search', label: '库存明细查询', icon: '◦', active: false },
      { id: 'inventory-check', label: '库存盘点记录', icon: '◦', active: false },
      { id: 'stock-material', label: '存量物资管理', icon: '◦', active: false },
    ],
  },
  {
    title: '资源全景可视',
    icon: '◎',
    expanded: true,
    items: [{ id: 'resource-view', label: '资源全景可视', icon: '◦', active: false }],
  },
])

const openTabs = ref([
  { id: 'material-ledger', label: '材料站台账', active: true },
  { id: 'material-approval', label: '材料站审批', active: false },
  { id: 'inventory-search', label: '库存明细查询', active: false },
  { id: 'resource-view', label: '资源全景可视', active: false },
])

const warehouseNatureOptions = ['变电站', '项目现场', '地市公司库房', '其他']
const approvalStatusOptions = ['待审批', '已通过', '已驳回']
const warehouseStatusOptions = ['启用', '停用', '-']
const specialtyOptions = ['发展', '安监', '设备营销', '物资', '调度', '数字化']
const firstUnitOptions = [
  '本部',
  '国网沈阳供电公司',
  '国网大连供电公司',
  '国网抚顺供电公司',
  '国网辽宁管培中心',
  '国网辽宁超高压公司',
  '国网辽宁建设分公司',
]

const tableColumns = [
  { key: 'index', label: '序号', width: '72px', sortable: false },
  { key: 'firstUnit', label: '一级单位', width: '170px', sortable: true },
  { key: 'secondUnit', label: '二级单位/部门', width: '170px', sortable: true },
  { key: 'thirdDepartment', label: '三级部门', width: '150px', sortable: true },
  { key: 'specialty', label: '所属专业', width: '110px', sortable: true },
  { key: 'nature', label: '仓库性质', width: '110px', sortable: true },
  { key: 'fullName', label: '仓库全称', width: '180px', sortable: true },
  { key: 'code', label: '仓库编码', width: '130px', sortable: true },
  { key: 'project', label: '关联项目', width: '150px', sortable: true },
  { key: 'provinceCityCounty', label: '省/市/县', width: '150px', sortable: true },
  { key: 'address', label: '仓库地址', width: '240px', sortable: true },
  { key: 'locationInfo', label: '位置信息', width: '160px', sortable: true },
  { key: 'coordinates', label: '经纬度', width: '140px', sortable: true },
  { key: 'note', label: '备注', width: '190px', sortable: true },
  { key: 'approvalStatus', label: '审批状态', width: '110px', sortable: true },
  { key: 'approver', label: '审批人', width: '100px', sortable: true },
  { key: 'approverPhone', label: '审批人联系方式', width: '150px', sortable: true },
  { key: 'approvalTime', label: '审批时间', width: '160px', sortable: true },
  { key: 'warehouseStatus', label: '仓库状态', width: '110px', sortable: true },
  { key: 'registrar', label: '注册人', width: '100px', sortable: true },
  { key: 'registrarPhone', label: '注册人联系方式', width: '150px', sortable: true },
  { key: 'registrationTime', label: '注册时间', width: '160px', sortable: true },
  { key: 'actions', label: '操作', width: '180px', sortable: false },
]

const toggleColumnKeys = tableColumns
  .map((column) => column.key)
  .filter((key) => key !== 'index' && key !== 'actions')

const columnVisibility = reactive(
  Object.fromEntries(toggleColumnKeys.map((key) => [key, true])),
)
const columnPickerOpen = ref(false)
const columnPickerRef = ref(null)

const warehouseRows = [
  {
    company: '华东建工集团',
    department: '物资设备部',
    nature: '变电站',
    fullName: 'G312改扩建项目临时材料站',
    code: 'WH-312-001',
    project: 'G312改扩建项目',
    provinceCityCounty: '辽宁省 / 沈阳市 / 和平区',
    address: '辽宁省沈阳市和平区南三好街28号',
    locationInfo: '主出入口东侧 120m',
    coordinates: '31.1812, 120.6387',
    note: '钢筋与模板分区存放，夜间封闭管理。',
    approvalStatus: '已通过',
    approver: '刘海',
    approverPhone: '138-2014-8801',
    approvalTime: '2026-08-22 09:15',
    warehouseStatus: '启用',
    registrar: '王强',
    registrarPhone: '139-8842-0315',
    registrationTime: '2026-08-18 09:12',
    registrationDate: '2026-08-18',
  },
  {
    company: '中铁建设二局',
    department: '项目管理部',
    nature: '项目现场',
    fullName: '滨江综合体项目钢材周转仓',
    code: 'WH-BJ-014',
    project: '滨江综合体项目',
    provinceCityCounty: '辽宁省 / 大连市 / 甘井子区',
    address: '辽宁省大连市甘井子区华北路188号',
    locationInfo: '施工围挡内南侧',
    coordinates: '30.2031, 120.2122',
    note: '待现场踏勘结果回传。',
    approvalStatus: '待审批',
    approver: '陈敏',
    approverPhone: '137-9006-2208',
    approvalTime: '2026-09-01 14:40',
    warehouseStatus: '-',
    registrar: '赵琳',
    registrarPhone: '136-5044-1933',
    registrationTime: '2026-08-29 10:03',
    registrationDate: '2026-08-29',
  },
  {
    company: '南方路桥工程',
    department: '综合管理部',
    nature: '地市公司库房',
    fullName: '南环高速提升工程综合仓',
    code: 'WH-NH-220',
    project: '南环高速提升工程',
    provinceCityCounty: '辽宁省 / 鞍山市 / 铁东区',
    address: '辽宁省鞍山市铁东区千山中路38号',
    locationInfo: '靠近搅拌站北门',
    coordinates: '23.0407, 113.3985',
    note: '已完成货架加固，适配大宗材料收发。',
    approvalStatus: '已通过',
    approver: '周立',
    approverPhone: '135-1180-6679',
    approvalTime: '2026-08-14 16:20',
    warehouseStatus: '启用',
    registrar: '李珂',
    registrarPhone: '138-3214-5098',
    registrationTime: '2026-08-12 11:28',
    registrationDate: '2026-08-12',
  },
  {
    company: '北方城建集团',
    department: '成本合约部',
    nature: '其他',
    fullName: '城东保障房项目辅材仓',
    code: 'WH-CD-087',
    project: '城东保障房项目',
    provinceCityCounty: '辽宁省 / 抚顺市 / 顺城区',
    address: '辽宁省抚顺市顺城区新华大街68号',
    locationInfo: '东门安检岗右侧',
    coordinates: '36.6758, 117.0831',
    note: '结构件暂存区已完成标识上墙。',
    approvalStatus: '待审批',
    approver: '待分配',
    approverPhone: '—',
    approvalTime: '—',
    warehouseStatus: '-',
    registrar: '孙伟',
    registrarPhone: '137-0038-4412',
    registrationTime: '2026-08-31 08:49',
    registrationDate: '2026-08-31',
  },
  {
    company: '华东建工集团',
    department: '物资设备部',
    nature: '变电站',
    fullName: '苏州工业园周材仓',
    code: 'WH-SZ-116',
    project: 'G312改扩建项目',
    provinceCityCounty: '辽宁省 / 锦州市 / 太和区',
    address: '辽宁省锦州市太和区凌西大街66号',
    locationInfo: '临时围挡北侧',
    coordinates: '31.2867, 120.7418',
    note: '支持夜间出入库，需提前报备。',
    approvalStatus: '已驳回',
    approver: '刘海',
    approverPhone: '138-2014-8801',
    approvalTime: '2026-08-26 17:10',
    warehouseStatus: '-',
    registrar: '王强',
    registrarPhone: '139-8842-0315',
    registrationTime: '2026-08-21 15:18',
    registrationDate: '2026-08-21',
  },
  {
    company: '中铁建设二局',
    department: '项目管理部',
    nature: '项目现场',
    fullName: '滨江综合体木模板堆场',
    code: 'WH-BJ-021',
    project: '滨江综合体项目',
    provinceCityCounty: '辽宁省 / 营口市 / 站前区',
    address: '辽宁省营口市站前区渤海大街52号',
    locationInfo: '临水施工便道旁',
    coordinates: '30.2095, 120.2111',
    note: '木模板分级堆放，防潮布已铺设。',
    approvalStatus: '已通过',
    approver: '陈敏',
    approverPhone: '137-9006-2208',
    approvalTime: '2026-08-24 10:25',
    warehouseStatus: '启用',
    registrar: '赵琳',
    registrarPhone: '136-5044-1933',
    registrationTime: '2026-08-20 13:05',
    registrationDate: '2026-08-20',
  },
  {
    company: '南方路桥工程',
    department: '综合管理部',
    nature: '地市公司库房',
    fullName: '南环高速提升工程油料仓',
    code: 'WH-NH-233',
    project: '南环高速提升工程',
    provinceCityCounty: '辽宁省 / 盘锦市 / 兴隆台区',
    address: '辽宁省盘锦市兴隆台区石油大街108号',
    locationInfo: '独立防火分区',
    coordinates: '22.9340, 113.2623',
    note: '已配置防泄漏托盘与消防器材。',
    approvalStatus: '待审批',
    approver: '周立',
    approverPhone: '135-1180-6679',
    approvalTime: '2026-09-02 09:35',
    warehouseStatus: '-',
    registrar: '李珂',
    registrarPhone: '138-3214-5098',
    registrationTime: '2026-08-28 16:44',
    registrationDate: '2026-08-28',
  },
  {
    company: '北方城建集团',
    department: '物资设备部',
    nature: '其他',
    fullName: '城东保障房项目机电仓',
    code: 'WH-CD-091',
    project: '城东保障房项目',
    provinceCityCounty: '辽宁省 / 辽阳市 / 白塔区',
    address: '辽宁省辽阳市白塔区中华大街90号',
    locationInfo: '西侧装卸平台',
    coordinates: '36.1138, 120.4634',
    note: '机电设备分批入库，库位动态调整。',
    approvalStatus: '已通过',
    approver: '韩雪',
    approverPhone: '139-1208-6641',
    approvalTime: '2026-08-17 11:42',
    warehouseStatus: '启用',
    registrar: '孙伟',
    registrarPhone: '137-0038-4412',
    registrationTime: '2026-08-15 14:20',
    registrationDate: '2026-08-15',
  },
  {
    company: '华东建工集团',
    department: '综合管理部',
    nature: '变电站',
    fullName: '总部备用材料仓',
    code: 'WH-HQ-003',
    project: 'G312改扩建项目',
    provinceCityCounty: '辽宁省 / 葫芦岛市 / 连山区',
    address: '辽宁省葫芦岛市连山区龙湾大街76号',
    locationInfo: '园区东北角',
    coordinates: '31.9583, 118.8194',
    note: '作为应急储备仓，仅保留常备件。',
    approvalStatus: '待审批',
    approver: '待分配',
    approverPhone: '—',
    approvalTime: '—',
    warehouseStatus: '-',
    registrar: '王强',
    registrarPhone: '139-8842-0315',
    registrationTime: '2026-09-03 09:58',
    registrationDate: '2026-09-03',
  },
  {
    company: '中铁建设二局',
    department: '成本合约部',
    nature: '地市公司库房',
    fullName: '滨江综合体周转周材仓',
    code: 'WH-BJ-030',
    project: '滨江综合体项目',
    provinceCityCounty: '辽宁省 / 阜新市 / 细河区',
    address: '辽宁省阜新市细河区中华路118号',
    locationInfo: '装卸通道南侧',
    coordinates: '29.8576, 121.5270',
    note: '与第三方物流共用场地，需预约进出。',
    approvalStatus: '已驳回',
    approver: '陈敏',
    approverPhone: '137-9006-2208',
    approvalTime: '2026-08-30 15:05',
    warehouseStatus: '-',
    registrar: '赵琳',
    registrarPhone: '136-5044-1933',
    registrationTime: '2026-08-27 10:11',
    registrationDate: '2026-08-27',
  },
  {
    company: '南方路桥工程',
    department: '项目管理部',
    nature: '项目现场',
    fullName: '南环高速提升工程砂石仓',
    code: 'WH-NH-241',
    project: '南环高速提升工程',
    provinceCityCounty: '辽宁省 / 本溪市 / 明山区',
    address: '辽宁省本溪市明山区解放北路42号',
    locationInfo: '料场东南角',
    coordinates: '23.2945, 113.2810',
    note: '砂石分仓管理，车辆排队区已规划。',
    approvalStatus: '已通过',
    approver: '周立',
    approverPhone: '135-1180-6679',
    approvalTime: '2026-08-19 13:48',
    warehouseStatus: '启用',
    registrar: '李珂',
    registrarPhone: '138-3214-5098',
    registrationTime: '2026-08-16 12:02',
    registrationDate: '2026-08-16',
  },
  {
    company: '北方城建集团',
    department: '综合管理部',
    nature: '其他',
    fullName: '城东保障房项目精装辅材仓',
    code: 'WH-CD-108',
    project: '城东保障房项目',
    provinceCityCounty: '辽宁省 / 丹东市 / 振兴区',
    address: '辽宁省丹东市振兴区锦山大街96号',
    locationInfo: '靠近临时办公区',
    coordinates: '36.6889, 117.1215',
    note: '精装辅材单独建账，便于领用追踪。',
    approvalStatus: '待审批',
    approver: '韩雪',
    approverPhone: '139-1208-6641',
    approvalTime: '2026-09-05 08:26',
    warehouseStatus: '-',
    registrar: '孙伟',
    registrarPhone: '137-0038-4412',
    registrationTime: '2026-09-01 16:09',
    registrationDate: '2026-09-01',
  },
  {
    company: '华东建工集团',
    department: '物资设备部',
    nature: '变电站',
    fullName: 'G312改扩建项目消防器材仓',
    code: 'WH-312-009',
    project: 'G312改扩建项目',
    provinceCityCounty: '辽宁省 / 沈阳市 / 浑南区',
    address: '辽宁省沈阳市浑南区创新路100号',
    locationInfo: '主仓库西翼',
    coordinates: '31.3895, 120.9542',
    note: '消防设备单独编码，月度巡检。',
    approvalStatus: '已通过',
    approver: '刘海',
    approverPhone: '138-2014-8801',
    approvalTime: '2026-08-13 15:33',
    warehouseStatus: '停用',
    registrar: '王强',
    registrarPhone: '139-8842-0315',
    registrationTime: '2026-08-10 10:19',
    registrationDate: '2026-08-10',
  },
  {
    company: '中铁建设二局',
    department: '项目管理部',
    nature: '项目现场',
    fullName: '滨江综合体机具周转仓',
    code: 'WH-BJ-044',
    project: '滨江综合体项目',
    provinceCityCounty: '辽宁省 / 大连市 / 金州区',
    address: '辽宁省大连市金州区金马路168号',
    locationInfo: '靠近塔吊作业区',
    coordinates: '30.1512, 120.2658',
    note: '机具周转频次高，采用电子台账登记。',
    approvalStatus: '待审批',
    approver: '待分配',
    approverPhone: '—',
    approvalTime: '—',
    warehouseStatus: '-',
    registrar: '赵琳',
    registrarPhone: '136-5044-1933',
    registrationTime: '2026-09-04 11:37',
    registrationDate: '2026-09-04',
  },
]

const createEmptyForm = () => ({
  fullName: '',
  project: '',
  nature: '',
  approvalStatus: '',
  firstUnit: '',
  secondUnit: '',
  thirdDepartment: '',
  specialty: '',
  warehouseStatus: '',
  registrationStart: '',
  registrationEnd: '',
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())
const currentPage = ref(1)
const pageSize = ref(8)
const sortState = ref({ key: 'registrationTime', direction: 'desc' })
const loading = ref(true)
const sidebarCollapsed = ref(false)
const filtersExpanded = ref(false)
const toast = ref(null)
const importInput = ref(null)

let loadingTimer = null
let toastTimer = null

const statusOrder = {
  待审批: 0,
  已通过: 1,
  已驳回: 2,
}

const warehouseOrder = {
  启用: 0,
  '-': 1,
  停用: 2,
}

function displayWarehouseStatus(row) {
  if (row.approvalStatus === '待审批' || row.approvalStatus === '已驳回') return '-'
  return row.warehouseStatus === '停用' ? '停用' : '启用'
}

const specialtyMap = {
  物资设备部: '物资',
  项目管理部: '调度',
  综合管理部: '数字化',
  成本合约部: '发展',
}

function rowThirdDepartment(row) {
  return `${row.department}仓储组`
}

function rowSpecialty(row) {
  return specialtyMap[row.department] || '安监'
}

function getColumnValue(row, key) {
  if (key === 'fullName') return rowWarehouseFullName(row)
  if (key === 'firstUnit') return rowFirstUnit(row)
  if (key === 'secondUnit') return rowSecondUnit(row)
  if (key === 'thirdDepartment') return rowThirdDepartment(row)
  if (key === 'specialty') return rowSpecialty(row)
  if (key === 'warehouseStatus') return displayWarehouseStatus(row)
  return row[key] ?? ''
}

function isSoftChipColumn(key) {
  return key === 'nature' || key === 'specialty'
}

function isStatusChipColumn(key) {
  return key === 'approvalStatus' || key === 'warehouseStatus'
}

const visibleTableColumns = computed(() =>
  tableColumns.filter(
    (column) => column.key === 'index' || column.key === 'actions' || columnVisibility[column.key] !== false,
  ),
)

const exportColumns = computed(() =>
  visibleTableColumns.value.filter((column) => column.key !== 'actions'),
)

function rowFirstUnit(row) {
  const rowIndex = warehouseRows.findIndex(({ code }) => code === row.code)
  return firstUnitOptions[Math.max(rowIndex, 0) % firstUnitOptions.length]
}

function rowSecondUnit(row) {
  return row.department
}

function rowWarehouseFullName(row) {
  return `${rowFirstUnit(row)}${rowThirdDepartment(row)}综合材料站`
}

function rowActionItems(row) {
  if (row.approvalStatus === '待审批' || row.approvalStatus === '已驳回') {
    return ['照片', '流程']
  }

  return displayWarehouseStatus(row) === '停用'
    ? ['更新', '启用', '照片', '流程']
    : ['更新', '停用', '照片', '流程']
}

const filteredRows = computed(() => {
  const filters = activeFilters.value

  return warehouseRows.filter((row) => {
    const textMatch = (source, query) =>
      !query || String(source).toLowerCase().includes(query.toLowerCase())

    const dateStartOk =
      !filters.registrationStart || row.registrationDate >= filters.registrationStart
    const dateEndOk = !filters.registrationEnd || row.registrationDate <= filters.registrationEnd

    return (
      textMatch(rowWarehouseFullName(row), filters.fullName) &&
      textMatch(row.project, filters.project) &&
      textMatch(rowFirstUnit(row), filters.firstUnit) &&
      textMatch(rowSecondUnit(row), filters.secondUnit) &&
      textMatch(rowThirdDepartment(row), filters.thirdDepartment) &&
      (!filters.nature || row.nature === filters.nature) &&
      (!filters.approvalStatus || row.approvalStatus === filters.approvalStatus) &&
      (!filters.specialty || rowSpecialty(row) === filters.specialty) &&
      (!filters.warehouseStatus || displayWarehouseStatus(row) === filters.warehouseStatus) &&
      dateStartOk &&
      dateEndOk
    )
  })
})

const sortedRows = computed(() => {
  const rows = [...filteredRows.value]
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

const pageCount = computed(() => Math.max(1, Math.ceil(filteredRows.value.length / pageSize.value)))
const paginatedRows = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  return sortedRows.value.slice(start, start + pageSize.value)
})
const startIndex = computed(() => (currentPage.value - 1) * pageSize.value)

function sortValue(row, key) {
  if (key === 'index') return row.code
  if (key === 'approvalStatus') return `${statusOrder[row.approvalStatus] ?? 99}-${row.approvalTime}`
  if (key === 'warehouseStatus') return `${warehouseOrder[displayWarehouseStatus(row)] ?? 99}-${row.code}`
  if (key === 'fullName') return rowWarehouseFullName(row)
  if (key === 'firstUnit') return rowFirstUnit(row)
  if (key === 'secondUnit') return rowSecondUnit(row)
  if (key === 'thirdDepartment') return rowThirdDepartment(row)
  if (key === 'specialty') return rowSpecialty(row)
  return row[key] ?? ''
}

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

function toggleGroup(group) {
  group.expanded = !group.expanded
}

function toggleFiltersExpanded() {
  filtersExpanded.value = !filtersExpanded.value
}

function toggleColumnPicker() {
  columnPickerOpen.value = !columnPickerOpen.value
}

function closeColumnPicker() {
  columnPickerOpen.value = false
}

function handleDocumentPointerDown(event) {
  if (!columnPickerOpen.value) return
  if (columnPickerRef.value?.contains(event.target)) return
  closeColumnPicker()
}

function openMenu(item) {
  if (!item.id) return
  const existing = openTabs.value.find(t => t.id === item.id)
  if (!existing) {
    openTabs.value.push({ id: item.id, label: item.label, active: false })
  }
  activateTab(item.id)
}

function activateTab(id) {
  openTabs.value = openTabs.value.map((tab) => ({
    ...tab,
    active: tab.id === id,
  }))
  menuGroups.forEach(g => {
    g.items.forEach(i => {
      i.active = i.id === id
    })
  })
}

const activeTabId = computed(() => openTabs.value.find(t => t.active)?.id || '')
const currentMenuLabel = computed(() => openTabs.value.find(t => t.active)?.label || '材料站台账')
const currentMenuGroupTitle = computed(() => {
  const activeId = activeTabId.value
  for (const group of menuGroups) {
    if (group.items.some(i => i.id === activeId)) return group.title
  }
  return '材料站管理'
})

function closeTab(id) {
  const currentTabs = openTabs.value
  const closeIndex = currentTabs.findIndex((tab) => tab.id === id)
  if (closeIndex === -1) return

  const wasActive = currentTabs[closeIndex]?.active
  const remaining = currentTabs.filter((tab) => tab.id !== id)

  if (!remaining.length) {
    openTabs.value = []
    return
  }

  if (wasActive && !remaining.some((tab) => tab.active)) {
    const nextIndex = Math.min(closeIndex, remaining.length - 1)
    remaining[nextIndex] = { ...remaining[nextIndex], active: true }
  }

  openTabs.value = remaining
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
  sortState.value = { key: 'registrationTime', direction: 'desc' }
  startLoading()
}

function startLoading() {
  loading.value = true
  window.clearTimeout(loadingTimer)
  loadingTimer = window.setTimeout(() => {
    loading.value = false
  }, 420)
}

function showToast(message, type = 'info') {
  toast.value = { message, type }
  window.clearTimeout(toastTimer)
  toastTimer = window.setTimeout(() => {
    toast.value = null
  }, 2200)
}

function handleImportClick() {
  importInput.value?.click()
}

function handleImportFile(event) {
  const [file] = event.target.files || []
  if (!file) return

  showToast(`已选择 ${file.name}，当前页面为示例台账，未接入后端导入`, 'success')
  event.target.value = ''
}

function downloadCsv() {
  const headers = exportColumns.value.map((column) => column.label)
  const rows = sortedRows.value.map((row, index) => [
    ...exportColumns.value.map((column) => {
      if (column.key === 'index') return index + 1
      if (column.key === 'actions') return ''
      return getColumnValue(row, column.key)
    }),
  ])

  const escape = (value) => `"${String(value ?? '').replaceAll('"', '""')}"`
  const csv = [headers, ...rows].map((line) => line.map(escape).join(',')).join('\n')
  const blob = new Blob(['\ufeff', csv], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  const stamp = new Date().toISOString().slice(0, 10).replaceAll('-', '')

  link.href = url
  link.download = `材料站台账_${stamp}.csv`
  link.click()
  window.setTimeout(() => URL.revokeObjectURL(url), 1000)
}

function handleExport() {
  if (!sortedRows.value.length) {
    showToast('当前没有可导出的数据', 'warning')
    return
  }

  downloadCsv()
  showToast('已开始导出当前筛选结果', 'success')
}

function handleRowAction(action, row) {
  showToast(`${action}：${rowWarehouseFullName(row)}`, 'info')
}

function handleSearch() {
  applyFilters()
  showToast(`已应用筛选，当前命中 ${filteredRows.value.length} 条`, 'success')
}

function handleReset() {
  resetFilters()
  showToast('已重置查询条件', 'info')
}

function gotoPage(page) {
  currentPage.value = Math.min(Math.max(1, page), pageCount.value)
}

onMounted(() => {
  loadingTimer = window.setTimeout(() => {
    loading.value = false
  }, 520)
  window.addEventListener('pointerdown', handleDocumentPointerDown)
})

onBeforeUnmount(() => {
  window.clearTimeout(loadingTimer)
  window.clearTimeout(toastTimer)
  window.removeEventListener('pointerdown', handleDocumentPointerDown)
})

function statusTone(value) {
  if (value === '已通过' || value === '启用') return 'success'
  if (value === '已驳回' || value === '停用') return 'danger'
  return 'neutral'
}

function sortIndicator(key) {
  if (sortState.value.key !== key) return '↕'
  return sortState.value.direction === 'asc' ? '↑' : '↓'
}
</script>

<template>
  <div class="shell" :class="{ 'shell--collapsed': sidebarCollapsed }">
    <aside class="sidebar">
      <div class="sidebar__brand">
        <button
          class="icon-button icon-button--ghost"
          type="button"
          :aria-label="sidebarCollapsed ? '展开侧边栏' : '折叠侧边栏'"
          @click="sidebarCollapsed = !sidebarCollapsed"
        >
          <span>{{ sidebarCollapsed ? '›' : '‹' }}</span>
        </button>
        <div v-if="!sidebarCollapsed" class="brand-copy">
          <strong>供应链运营平台</strong>
          <span>Enterprise SaaS Console</span>
        </div>
      </div>

      <nav class="sidebar__nav" aria-label="主导航">
        <section v-for="group in menuGroups" :key="group.title" class="nav-group">
          <button
            class="nav-group__title"
            type="button"
            :aria-expanded="group.expanded"
            :title="group.title"
            @click="toggleGroup(group)"
          >
            <span class="nav-group__badge">{{ group.icon }}</span>
            <span v-if="!sidebarCollapsed" class="nav-group__label">{{ group.title }}</span>
            <span v-if="!sidebarCollapsed" class="nav-group__chevron">
              {{ group.expanded ? '▾' : '▸' }}
            </span>
          </button>

          <div v-show="group.expanded" class="nav-group__items">
            <button
              v-for="item in group.items"
              :key="item.id"
              type="button"
              class="nav-item"
              :class="{ 'nav-item--active': item.active, 'nav-item--child': true }"
              :title="item.label"
              @click="openMenu(item)"
            >
              <span class="nav-item__badge">{{ item.icon }}</span>
              <span v-if="!sidebarCollapsed">{{ item.label }}</span>
            </button>
          </div>
        </section>
      </nav>
    </aside>

    <div class="workspace">
      <header class="topbar">
        <div class="breadcrumb">
          <span>首页</span>
          <span>{{ currentMenuGroupTitle }}</span>
          <span>{{ currentMenuLabel }}</span>
        </div>

        <div class="topbar__meta">
          <button class="icon-button" type="button" aria-label="通知">
            <span>◌</span>
          </button>
          <button class="icon-button" type="button" aria-label="帮助">
            <span>?</span>
          </button>
          <div class="avatar">SC</div>
        </div>
      </header>

      <div class="tabs">
        <div
          v-for="(tab, index) in openTabs"
          :key="tab.id"
          class="tab"
          :class="{ 'tab--active': tab.active }"
          role="button"
          tabindex="0"
          @click="activateTab(tab.id)"
          @keydown.enter.prevent="activateTab(tab.id)"
          @keydown.space.prevent="activateTab(tab.id)"
        >
          <span class="tab__label">{{ tab.label }}</span>
          <button
            class="tab__close"
            type="button"
            :aria-label="`关闭 ${tab.label}`"
            @click.stop="closeTab(tab.id)"
          >
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="M6 6l12 12"></path>
              <path d="M18 6L6 18"></path>
            </svg>
          </button>
          <div v-if="!tab.active && (index === openTabs.length - 1 || !openTabs[index + 1].active)" class="tab__divider"></div>
        </div>
      </div>

      <main class="content">
        <div v-show="activeTabId === 'material-ledger'" style="display: contents;">
          <section class="panel panel--filters">
          <form
            class="filter-grid"
            :class="{ 'filter-grid--collapsed': !filtersExpanded }"
            @submit.prevent="handleSearch"
          >
            <label class="field">
              <span>仓库全称</span>
              <input v-model.trim="form.fullName" type="text" placeholder="请输入仓库全称" @keydown.enter.prevent="handleSearch" />
            </label>

            <label class="field">
              <span>仓库状态</span>
              <select v-model="form.warehouseStatus">
                <option value="">全部状态</option>
                <option v-for="option in warehouseStatusOptions" :key="option" :value="option">{{ option }}</option>
              </select>
            </label>

            <label class="field">
              <span>审批状态</span>
              <select v-model="form.approvalStatus">
                <option value="">全部状态</option>
                <option v-for="option in approvalStatusOptions" :key="option" :value="option">{{ option }}</option>
              </select>
            </label>

            <label v-show="filtersExpanded" class="field field--advanced">
              <span>仓库性质</span>
              <select v-model="form.nature">
                <option value="">全部性质</option>
                <option v-for="option in warehouseNatureOptions" :key="option" :value="option">{{ option }}</option>
              </select>
            </label>

            <label v-show="filtersExpanded" class="field field--advanced">
              <span>一级单位</span>
              <input v-model.trim="form.firstUnit" type="text" placeholder="请输入一级单位" @keydown.enter.prevent="handleSearch" />
            </label>

            <label v-show="filtersExpanded" class="field field--advanced">
              <span>二级单位/部门</span>
              <input v-model.trim="form.secondUnit" type="text" placeholder="请输入二级单位/部门" @keydown.enter.prevent="handleSearch" />
            </label>

            <label v-show="filtersExpanded" class="field field--advanced">
              <span>三级部门</span>
              <input v-model.trim="form.thirdDepartment" type="text" placeholder="请输入三级部门" @keydown.enter.prevent="handleSearch" />
            </label>

            <label v-show="filtersExpanded" class="field field--advanced">
              <span>所属专业</span>
              <select v-model="form.specialty">
                <option value="">全部专业</option>
                <option v-for="option in specialtyOptions" :key="option" :value="option">{{ option }}</option>
              </select>
            </label>

            <label v-show="filtersExpanded" class="field field--advanced">
              <span>关联项目</span>
              <input v-model.trim="form.project" type="text" placeholder="请输入关联项目" @keydown.enter.prevent="handleSearch" />
            </label>

            <div v-show="filtersExpanded" class="field field--advanced">
              <span>注册时间</span>
              <div class="range-inputs">
                <input v-model="form.registrationStart" type="date" />
                <span>至</span>
                <input v-model="form.registrationEnd" type="date" />
              </div>
            </div>

            <div class="filter-actions">
              <button class="btn btn--primary" type="submit">
                <span class="btn__icon btn__icon--light" aria-hidden="true">
                  <svg viewBox="0 0 24 24" aria-hidden="true">
                    <circle cx="11" cy="11" r="6.5"></circle>
                    <path d="M16.5 16.5L21 21"></path>
                  </svg>
                </span>
                查询
              </button>
              <button class="btn btn--ghost" type="button" @click="handleReset">
                <span class="btn__icon" aria-hidden="true">
                  <svg viewBox="0 0 24 24" aria-hidden="true">
                    <path d="M4 12a8 8 0 1 0 2.34-5.66"></path>
                    <path d="M4 4v5h5"></path>
                  </svg>
                </span>
                重置
              </button>
              <button
                class="filter-toggle"
                type="button"
                :aria-expanded="filtersExpanded"
                @click="toggleFiltersExpanded"
              >
                {{ filtersExpanded ? '收起' : '更多' }}
              </button>
            </div>
          </form>
        </section>

        <section class="panel panel--results">
          <div class="results-bar">
            <div class="toolbar">
              <button class="btn btn--ghost" type="button" @click="handleImportClick">
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

                <div v-if="columnPickerOpen" class="column-picker__panel">
                  <div class="column-picker__title">列筛选</div>
                  <label
                    v-for="column in tableColumns.filter((item) => item.key !== 'index' && item.key !== 'actions')"
                    :key="column.key"
                    class="column-picker__option"
                  >
                    <input v-model="columnVisibility[column.key]" class="column-picker__checkbox" type="checkbox" />
                    <span>{{ column.label }}</span>
                  </label>
                </div>
              </div>
            </div>
          </div>

          <div class="table-wrap">
            <table class="data-table">
              <thead>
                <tr>
                  <th
                    v-for="column in visibleTableColumns"
                    :key="column.key"
                    :style="{ minWidth: column.width }"
                    :class="[
                      column.sortable && 'is-sortable',
                      sortState.key === column.key && 'is-sorted',
                      column.key === 'actions' && 'cell--actions',
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
                  <td
                    v-for="column in visibleTableColumns"
                    :key="column.key"
                    :class="[
                      column.key === 'index' && 'cell--index',
                      column.key === 'actions' && 'cell--actions',
                    ]"
                  >
                    <span class="skeleton-line" :class="`skeleton-line--${column.key}`"></span>
                  </td>
                </tr>
              </tbody>

              <tbody v-else-if="paginatedRows.length">
                <tr v-for="(row, index) in paginatedRows" :key="`${row.code}-${row.registrationTime}`">
                  <template v-for="column in visibleTableColumns" :key="column.key">
                    <td v-if="column.key === 'index'" class="cell--index">
                      {{ startIndex + index + 1 }}
                    </td>
                    <td v-else-if="column.key === 'actions'" class="actions cell--actions">
                      <button
                        v-for="action in rowActionItems(row)"
                        :key="action"
                        class="text-button"
                        type="button"
                        @click="handleRowAction(action, row)"
                      >
                        {{ action }}
                      </button>
                    </td>
                    <td v-else-if="isSoftChipColumn(column.key)">
                      <span class="chip chip--soft">{{ getColumnValue(row, column.key) }}</span>
                    </td>
                    <td v-else-if="isStatusChipColumn(column.key)">
                      <span class="chip" :class="`chip--${statusTone(getColumnValue(row, column.key))}`">
                        {{ getColumnValue(row, column.key) }}
                      </span>
                    </td>
                    <td v-else :title="getColumnValue(row, column.key)">
                      {{ getColumnValue(row, column.key) }}
                    </td>
                  </template>
                </tr>
              </tbody>

              <tbody v-else>
                <tr>
                  <td :colspan="visibleTableColumns.length">
                    <div class="empty-state">
                      <div class="empty-state__icon">□</div>
                      <strong>暂无匹配数据</strong>
                      <p>请调整查询条件后重新检索。</p>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>

            <div v-if="loading" class="loading-mask">
              <div class="loading-card">
                <div class="spinner"></div>
                <p>正在加载台账数据…</p>
              </div>
            </div>
          </div>

          <div class="table-footer">
            <div class="page-size">
              <span style="margin-right: 8px;">共 {{ filteredRows.length }} 条</span>
              每页
              <select v-model.number="pageSize" @change="gotoPage(1)">
                <option :value="8">8 条</option>
                <option :value="10">10 条</option>
                <option :value="20">20 条</option>
              </select>
            </div>

            <div class="pagination">
              <button class="page-btn" type="button" :disabled="currentPage === 1" @click="gotoPage(currentPage - 1)">
                ‹
              </button>

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

              <button
                class="page-btn"
                type="button"
                :disabled="currentPage === pageCount"
                @click="gotoPage(currentPage + 1)"
              >
                ›
              </button>
            </div>
          </div>
        </section>
        </div>
        <BusinessApproval v-if="activeTabId === 'business-approval'" />
        <MaterialStationApproval v-if="activeTabId === 'material-approval'" />
      </main>
    </div>

    <transition name="toast-fade">
      <div v-if="toast" class="toast" :class="`toast--${toast.type}`">
        {{ toast.message }}
      </div>
    </transition>

    <input
      ref="importInput"
      class="sr-only"
      type="file"
      accept=".csv,.xls,.xlsx"
      @change="handleImportFile"
    />
  </div>
</template>
