<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'

const createEmptyForm = () => ({
  account: '',
  name: '',
  phone: '',
  status: ''
})

const form = reactive(createEmptyForm())
const activeFilters = ref(createEmptyForm())

const statusOptions = ['正常', '冻结']

const tableColumns = ref([
  { key: 'index', label: '序号', width: '60px', sortable: false, visible: true },
  { key: 'account', label: '账号', width: '120px', sortable: true, visible: true },
  { key: 'name', label: '姓名', width: '100px', sortable: true, visible: true },
  { key: 'avatar', label: '头像', width: '80px', sortable: false, visible: true },
  { key: 'gender', label: '性别', width: '80px', sortable: true, visible: true },
  { key: 'birthday', label: '生日', width: '120px', sortable: true, visible: true },
  { key: 'phone', label: '手机号码', width: '140px', sortable: true, visible: true },
  { key: 'department', label: '部门', width: '280px', sortable: true, visible: true },
  { key: 'roles', label: '角色', width: '180px', sortable: true, visible: true },
  { key: 'status', label: '状态', width: '100px', sortable: true, visible: true },
  { key: 'actions', label: '操作', width: '320px', sortable: false, visible: true }
])

const generateMockData = () => {
  const firstUnits = ['国网沈阳供电公司', '国网大连供电公司', '国网抚顺供电公司']
  const secondUnits = ['铁西分公司', '甘井子分公司', '顺城分公司']
  const thirdDepartments = ['研发部', '市场部', '财务部', '运检部', '物资部']
  const realNames = [
    '张伟', '王芳', '李娜', '赵强', '陈明', '刘洋', '杨静', '黄勇', '周杰', '吴艳',
    '徐磊', '孙超', '马丽', '朱涛', '胡娟', '林峰', '郭鑫', '何平', '高翔', '郑洁',
    '罗浩', '宋婷', '唐辉', '韩雪', '曹宇'
  ]
  const pinyinAccounts = [
    'zhangwei', 'wangfang', 'lina', 'zhaoqiang', 'chenming', 'liuyang', 'yangjing', 'huangyong', 'zhoujie', 'wuyan',
    'xulei', 'sunchao', 'mali', 'zhutao', 'hujuan', 'linfeng', 'guoxin', 'heping', 'gaoxiang', 'zhengjie',
    'luohao', 'songting', 'tanghui', 'hanxue', 'caoyu'
  ]

  return Array.from({ length: 25 }, (_, i) => {
    const isFrozen = i % 4 === 0
    return {
      id: i + 1,
      account: pinyinAccounts[i % pinyinAccounts.length],
      name: realNames[i % realNames.length],
      avatar: `https://api.dicebear.com/7.x/avataaars/svg?seed=${i}`,
      gender: i % 2 === 0 ? '男' : '女',
      birthday: `199${i % 10}-0${(i % 9) + 1}-1${i % 9}`,
      phone: `138${String(i).padStart(8, '0')}`,
      department: `${firstUnits[i % firstUnits.length]} / ${secondUnits[i % secondUnits.length]} / ${thirdDepartments[i % thirdDepartments.length]}`,
      roles: i % 2 === 0 ? ['系统管理员', '审批员'] : ['普通用户'],
      status: isFrozen ? '冻结' : '正常',
      description: `这是一个很长的描述文字，用于测试两行截断显示效果。这里包含了很多文字内容，如果超过两行就会显示省略号。第${i}条测试数据。` // Extra text to test multiline
    }
  })
}

const mockData = generateMockData()

const currentPage = ref(1)
const pageSize = ref(10)
const loading = ref(false)
let loadingTimer = null

const isEditDrawerVisible = ref(false)
const editingUser = reactive({
  account: '', name: '', employeeId: '', wmsUsername: '',
  jobTitle: '', specialty: '', roles: [], department: '',
  tenant: '', identity: '', avatar: '', birthday: '', gender: '',
  email: '', phone: '', factory: '', materialStation: ''
})

const specialtyOptions = ['计划', '采购', '合同', '供应', '质量', '仓储', '督察', '应急', '废旧']
const roleOptions = ['系统管理员', '普通用户', '审批员']
const tenantOptions = ['租户A', '租户B', '租户C']

const filteredRows = computed(() => {
  let rows = mockData
  const f = activeFilters.value
  
  if (f.account) rows = rows.filter(r => r.account.includes(f.account))
  if (f.name) rows = rows.filter(r => r.name.includes(f.name))
  if (f.phone) rows = rows.filter(r => r.phone.includes(f.phone))
  if (f.status) rows = rows.filter(r => r.status === f.status)
  
  return rows
})

const sortState = ref({ key: 'account', direction: 'asc' })

function sortValue(row, key) {
  if (key === 'roles') return row.roles.join('、')
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

onMounted(() => {
  startLoading()
})

onBeforeUnmount(() => {
  window.clearTimeout(loadingTimer)
})

function handleAddUser() {
  alert('添加用户功能开发中')
}

function handleExport() {
  alert('导出功能开发中')
}

function handleRecycleBin() {
  alert('回收站功能开发中')
}

function handleAction(action, row) {
  if (action === '编辑') {
    openEditDrawer(row)
  } else {
    alert(`对 ${row.name} 触发操作：${action}`)
  }
}

function openEditDrawer(row) {
  Object.assign(editingUser, {
    account: row.account || '',
    name: row.name || '',
    employeeId: row.id ? String(row.id).padStart(4, '0') : '',
    wmsUsername: '',
    jobTitle: '',
    specialty: '',
    roles: Array.isArray(row.roles) ? [...row.roles] : [],
    department: row.department || '',
    tenant: '',
    identity: '普通用户',
    avatar: row.avatar || '',
    birthday: row.birthday || '',
    gender: row.gender || '男',
    email: '',
    phone: row.phone || '',
    factory: '',
    materialStation: ''
  })
  isEditDrawerVisible.value = true
}

function closeEditDrawer() {
  isEditDrawerVisible.value = false
}

function saveEdit() {
  alert('保存成功')
  closeEditDrawer()
}

// --- 材料站专业维护弹窗逻辑 ---
const isSpecialtyModalVisible = ref(false)
const allSpecialtiesBase = [
  '发展', '安监', '设备', '营销', '物资', '调度', '数字化', '办公室', '财务',
  '法律', '基建', '组织', '党建', '宣传', '纪委', '科技', '企管', '审计',
  '后勤', '工会', '交易', '配网', '调试', '产业发展', '供服', '其他专业', '无专业'
]
const leftSpecialties = ref([])
const rightSpecialties = ref([])

function openSpecialtyModal() {
  const currentAuthorized = editingUser.materialStation
    ? editingUser.materialStation.split('、')
    : []

  rightSpecialties.value = currentAuthorized.map(name => ({ name, checked: false }))
  leftSpecialties.value = allSpecialtiesBase
    .filter(name => !currentAuthorized.includes(name))
    .map(name => ({ name, checked: false }))

  isSpecialtyModalVisible.value = true
}

function closeSpecialtyModal() {
  isSpecialtyModalVisible.value = false
}

function authorizeSelected() {
  const toMove = leftSpecialties.value.filter(item => item.checked)
  leftSpecialties.value = leftSpecialties.value.filter(item => !item.checked)
  rightSpecialties.value.push(...toMove.map(item => ({ ...item, checked: false })))
}

function revokeSelected() {
  const toMove = rightSpecialties.value.filter(item => item.checked)
  rightSpecialties.value = rightSpecialties.value.filter(item => !item.checked)
  leftSpecialties.value.push(...toMove.map(item => ({ ...item, checked: false })))
}

function authorizeAll() {
  rightSpecialties.value.push(...leftSpecialties.value.map(item => ({ ...item, checked: false })))
  leftSpecialties.value = []
}

function revokeAll() {
  leftSpecialties.value.push(...rightSpecialties.value.map(item => ({ ...item, checked: false })))
  rightSpecialties.value = []
}

function confirmSpecialtyModal() {
  editingUser.materialStation = rightSpecialties.value.map(item => item.name).join('、')
  closeSpecialtyModal()
}
// --- 结束 ---

function getCellValue(row, key) {
  if (key === 'roles') {
    return Array.isArray(row[key]) ? row[key].join('、') : row[key]
  }
  return row[key]
}

</script>

<template>
  <div style="display: contents;">
    <section class="panel panel--filters">
      <form class="filter-grid filter-grid--collapsed" @submit.prevent="applyFilters">
        <label class="field">
          <span>账号</span>
          <input v-model.trim="form.account" type="text" placeholder="请输入账号" />
        </label>
        <label class="field">
          <span>姓名</span>
          <input v-model.trim="form.name" type="text" placeholder="请输入姓名" />
        </label>
        <label class="field">
          <span>手机号码</span>
          <input v-model.trim="form.phone" type="text" placeholder="请输入手机号码" />
        </label>
        <label class="field">
          <span>用户状态</span>
          <select v-model="form.status">
            <option value="">全部</option>
            <option v-for="opt in statusOptions" :key="opt" :value="opt">{{ opt }}</option>
          </select>
        </label>

        <div class="filter-actions" style="grid-column: 1 / -1;">
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
          <button class="btn btn--primary" type="button" @click="handleAddUser">
            添加用户
          </button>
          <button class="btn btn--ghost" type="button" @click="handleExport">
            <span class="btn__icon">↓</span>
            导出
          </button>
          <button class="btn btn--ghost" type="button" @click="handleRecycleBin">
            回收站
          </button>
        </div>
      </div>
      
      <div class="table-wrap" style="overflow-x: auto;">
        <table class="data-table" style="min-width: 1400px; table-layout: fixed;">
          <thead>
            <tr>
              <th
                v-for="column in tableColumns"
                :key="column.key"
                :style="{ minWidth: column.width, width: column.width, maxWidth: column.width, boxSizing: 'border-box', textAlign: 'left' }"
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
              <td v-for="column in tableColumns" :key="column.key" :class="{ 'cell--actions': column.key === 'actions' }" :style="column.key === 'actions' ? { minWidth: column.width, width: column.width, maxWidth: column.width, boxSizing: 'border-box' } : {}">
                <span class="skeleton-line"></span>
              </td>
            </tr>
          </tbody>
          <tbody v-else-if="paginatedRows.length">
            <tr v-for="(row, index) in paginatedRows" :key="row.id">
              <template v-for="column in tableColumns" :key="column.key">
                <td v-if="column.key === 'index'" class="cell--index">
                  {{ startIndex + index + 1 }}
                </td>
                <td v-else-if="column.key === 'avatar'">
                  <img :src="row.avatar" alt="avatar" style="width: 32px; height: 32px; border-radius: 50%; object-fit: cover; display: block;" />
                </td>
                <td v-else-if="column.key === 'status'">
                  <span class="chip" :class="row.status === '正常' ? 'chip--success' : 'chip--warning'">{{ row.status }}</span>
                </td>
                <td v-else-if="column.key === 'actions'" class="actions cell--actions" :style="{ width: column.width, minWidth: column.width, maxWidth: column.width, boxSizing: 'border-box', whiteSpace: 'nowrap' }">
                  <button type="button" class="text-button" @click="handleAction('编辑', row)">编辑</button>
                  <button type="button" class="text-button" @click="handleAction('详情', row)">详情</button>
                  <button type="button" class="text-button" @click="handleAction('密码', row)">密码</button>
                  <button type="button" class="text-button" @click="handleAction('删除', row)" style="color: var(--danger);">删除</button>
                  <button type="button" class="text-button" @click="handleAction(row.status === '冻结' ? '解锁' : '冻结', row)">
                    {{ row.status === '冻结' ? '解锁' : '冻结' }}
                  </button>
                  <button type="button" class="text-button" @click="handleAction('签章', row)">签章</button>
                </td>
                <td v-else>
                  <div class="multi-line-cell" :title="getCellValue(row, column.key)">
                    {{ getCellValue(row, column.key) || '-' }}
                  </div>
                </td>
              </template>
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

    <!-- 编辑抽屉页面 -->
    <div v-if="isEditDrawerVisible" class="drawer-overlay" @click="closeEditDrawer">
      <div class="drawer-container" @click.stop>
        <div class="drawer-header">
          <h3>编辑用户</h3>
          <button class="close-btn" @click="closeEditDrawer">&times;</button>
        </div>
        <div class="drawer-body">
          <form class="edit-form" @submit.prevent="saveEdit">
            <div class="form-group required">
              <label>账号</label>
              <input type="text" v-model="editingUser.account" placeholder="请输入账号" required />
            </div>
            <div class="form-group required">
              <label>姓名</label>
              <input type="text" v-model="editingUser.name" placeholder="请输入姓名" required />
            </div>
            <div class="form-group required">
              <label>工号</label>
              <input type="text" v-model="editingUser.employeeId" placeholder="请输入工号" required />
            </div>
            <div class="form-group">
              <label>wms用户名</label>
              <input type="text" v-model="editingUser.wmsUsername" placeholder="请输入wms用户名" />
            </div>
            <div class="form-group">
              <label>职务</label>
              <div class="popup-select">
                <input type="text" readonly :value="editingUser.jobTitle" placeholder="请选择职务" />
                <button type="button" class="btn btn--small">选择</button>
              </div>
            </div>
            <div class="form-group">
              <label>专业</label>
              <select v-model="editingUser.specialty">
                <option value="">请选择专业</option>
                <option v-for="opt in specialtyOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>角色分配</label>
              <select v-model="editingUser.roles" multiple class="multi-select">
                <option v-for="opt in roleOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>部门分配</label>
              <div class="popup-select">
                <input type="text" readonly :value="editingUser.department" placeholder="请选择部门" />
                <button type="button" class="btn btn--small">选择</button>
              </div>
            </div>
            <div class="form-group">
              <label>租户分配</label>
              <select v-model="editingUser.tenant">
                <option value="">请选择租户</option>
                <option v-for="opt in tenantOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>身份</label>
              <select v-model="editingUser.identity">
                <option value="普通用户">普通用户</option>
                <option value="领导">领导</option>
              </select>
            </div>
            <div class="form-group">
              <label>头像</label>
              <div class="avatar-upload">
                <img v-if="editingUser.avatar" :src="editingUser.avatar" class="avatar-preview" alt="avatar" />
                <button type="button" class="btn btn--small">上传照片</button>
              </div>
            </div>
            <div class="form-group">
              <label>生日</label>
              <input type="date" v-model="editingUser.birthday" />
            </div>
            <div class="form-group">
              <label>性别</label>
              <select v-model="editingUser.gender">
                <option value="男">男</option>
                <option value="女">女</option>
              </select>
            </div>
            <div class="form-group">
              <label>邮箱</label>
              <input type="email" v-model="editingUser.email" placeholder="请输入邮箱" />
            </div>
            <div class="form-group">
              <label>手机号码</label>
              <input type="tel" v-model="editingUser.phone" placeholder="请输入手机号码" />
            </div>
            <div class="form-group full-width">
              <label>工厂仓库</label>
              <div class="popup-box">
                <div class="popup-box-header">
                  <button type="button" class="btn btn--primary btn--small">维护</button>
                </div>
                <div class="popup-box-content" v-if="!editingUser.factory">
                  <img src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='100' height='100' viewBox='0 0 100 100'><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' fill='%23999'>暂无数据</text></svg>" alt="暂无数据" class="placeholder-img" />
                </div>
                <div class="popup-box-content" v-else>
                  {{ editingUser.factory }}
                </div>
              </div>
            </div>
            <div class="form-group full-width">
              <label>材料站专业</label>
              <div class="popup-box">
                <div class="popup-box-header">
                  <button type="button" class="btn btn--primary btn--small" @click="openSpecialtyModal">维护</button>
                </div>
                <div class="popup-box-content" v-if="!editingUser.materialStation">
                  <img src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='100' height='100' viewBox='0 0 100 100'><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' fill='%23999'>查看全部专业</text></svg>" alt="查看全部专业" class="placeholder-img" />
                </div>
                <div class="popup-box-content text-content" v-else>
                  {{ editingUser.materialStation }}
                </div>
              </div>
            </div>
          </form>
        </div>
        <div class="drawer-footer">
          <button class="btn btn--ghost" @click="closeEditDrawer">取消</button>
          <button class="btn btn--primary" @click="saveEdit">保存</button>
        </div>
      </div>
    </div>

    <!-- 材料站专业维护弹窗 -->
    <div v-if="isSpecialtyModalVisible" class="modal-overlay" @click="closeSpecialtyModal">
      <div class="modal-container" @click.stop>
        <div class="modal-header">
          <h3>材料站专业维护</h3>
          <button class="close-btn" @click="closeSpecialtyModal">&times;</button>
        </div>
        <div class="modal-body transfer-container">
          <!-- 左侧列表 -->
          <div class="transfer-panel">
            <div class="transfer-panel-header">
              全部专业 (<span class="text-orange">{{ leftSpecialties.length }}</span>)
            </div>
            <div class="transfer-list">
              <label v-for="(item, index) in leftSpecialties" :key="'left-'+index" class="transfer-item">
                <input type="checkbox" v-model="item.checked" />
                <span>{{ item.name }}</span>
              </label>
            </div>
          </div>

          <!-- 中间按钮 -->
          <div class="transfer-actions">
            <button type="button" class="btn btn--primary btn--small" @click="authorizeSelected" :disabled="!leftSpecialties.some(i => i.checked)">授权 &gt;</button>
            <button type="button" class="btn btn--primary btn--small" @click="authorizeAll" :disabled="leftSpecialties.length === 0">全部授权 &gt;&gt;</button>
            <button type="button" class="btn btn--ghost btn--small" @click="revokeSelected" :disabled="!rightSpecialties.some(i => i.checked)">&lt; 取消授权</button>
            <button type="button" class="btn btn--ghost btn--small" @click="revokeAll" :disabled="rightSpecialties.length === 0">&lt;&lt; 全部取消</button>
          </div>

          <!-- 右侧列表 -->
          <div class="transfer-panel">
            <div class="transfer-panel-header">
              已授权专业 (<span class="text-orange">{{ rightSpecialties.length }}</span>)
            </div>
            <div class="transfer-list">
              <label v-for="(item, index) in rightSpecialties" :key="'right-'+index" class="transfer-item">
                <input type="checkbox" v-model="item.checked" />
                <span>{{ item.name }}</span>
              </label>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn btn--ghost" @click="closeSpecialtyModal">取消</button>
          <button class="btn btn--primary" @click="confirmSpecialtyModal">确认</button>
        </div>
      </div>
    </div>
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

.multi-line-cell {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
  word-break: break-all;
  white-space: normal;
  line-height: 1.5;
  max-height: 3em; /* 2 times line height */
}

/* 确保操作列固定在最右侧 */
.cell--actions {
  position: sticky;
  right: 0;
  background-color: #fff;
  z-index: 1;
}

/* 斑马纹和 hover 状态下需要改变 sticky 单元格的背景色以匹配，假设 data-table 有 hover 效果 */
.data-table tbody tr:hover .cell--actions {
  background-color: #f8fafc; /* 根据项目实际 hover 颜色调整，通常是比较浅的颜色 */
}

/* 抽屉样式 */
.drawer-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  display: flex;
  justify-content: flex-end;
}
.drawer-container {
  width: 600px;
  background: #fff;
  height: 100%;
  display: flex;
  flex-direction: column;
  box-shadow: -2px 0 8px rgba(0,0,0,0.15);
  animation: slideIn 0.3s ease-out;
}
@keyframes slideIn {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}
.drawer-header {
  padding: 16px 20px;
  border-bottom: 1px solid var(--line);
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.drawer-header h3 {
  margin: 0;
  font-size: 18px;
  color: var(--text-primary);
}
.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #999;
}
.close-btn:hover {
  color: #333;
}
.drawer-body {
  padding: 20px;
  flex: 1;
  overflow-y: auto;
}
.drawer-footer {
  padding: 16px 20px;
  border-top: 1px solid var(--line);
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}
.edit-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}
.form-group {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  gap: 16px;
  width: 100%;
}
.form-group.full-width {
  /* no longer needed */
}
.form-group label {
  width: 20%; /* 标签占整体宽度的 20% */
  text-align: right;
  line-height: 36px; /* 与右侧输入框垂直居中对齐 */
  font-size: 14px;
  color: var(--text-primary);
  flex-shrink: 0;
}
.form-group.required label::before {
  content: '*';
  color: var(--danger);
  margin-right: 4px;
}
/* 控制输入控件区占整体宽度的 70%，留出 10% 右侧空白 */
.form-group > input[type="text"],
.form-group > input[type="email"],
.form-group > input[type="tel"],
.form-group > input[type="date"],
.form-group > select,
.form-group > .popup-select,
.form-group > .radio-group,
.form-group > .avatar-upload,
.form-group > .popup-box {
  width: 70%;
  box-sizing: border-box;
}
.form-group input[type="text"],
.form-group input[type="email"],
.form-group input[type="tel"],
.form-group input[type="date"],
.form-group select {
  padding: 8px 12px;
  border: 1px solid var(--line);
  border-radius: 4px;
  font-size: 14px;
  outline: none;
  height: 36px;
}
.form-group input[type="text"]:focus,
.form-group input[type="email"]:focus,
.form-group input[type="tel"]:focus,
.form-group input[type="date"]:focus,
.form-group select:focus {
  border-color: var(--primary);
}
.multi-select {
  height: 80px !important;
}
.popup-select {
  display: flex;
  gap: 8px;
}
.popup-select input {
  flex: 1;
  background-color: #f9fafb;
  cursor: not-allowed;
}
.radio-group {
  display: flex;
  gap: 24px;
  align-items: center;
  height: 36px;
}
.radio-group label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 14px;
  color: var(--text-primary);
  position: relative;
  white-space: nowrap; /* 强制文字不换行，防止挤压 */
}
.radio-group input[type="radio"] {
  appearance: none;
  -webkit-appearance: none;
  width: 16px;
  height: 16px;
  min-width: 16px; /* 强制最小宽度，防止被挤压成椭圆 */
  min-height: 16px;
  border: 1px solid #dcdfe6;
  border-radius: 50%;
  outline: none;
  cursor: pointer;
  margin: 0;
  position: relative;
  transition: all 0.2s ease;
  box-sizing: border-box; /* 确保 padding/border 不影响尺寸 */
}
.radio-group input[type="radio"]:hover {
  border-color: var(--primary);
}
.radio-group input[type="radio"]:checked {
  border-color: var(--primary);
  background-color: #fff;
}
.radio-group input[type="radio"]:checked::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: var(--primary);
}
.avatar-upload {
  display: flex;
  align-items: center;
  gap: 12px;
}
.avatar-preview {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid var(--line);
}
.popup-box {
  border: 1px dashed var(--line);
  border-radius: 4px;
  padding: 12px;
}
.popup-box-header {
  margin-bottom: 12px;
  display: flex;
  justify-content: flex-end;
}
.btn--small {
  padding: 4px 12px;
  font-size: 12px;
  height: 28px;
  line-height: 1;
}
.popup-box-content {
  display: flex;
  justify-content: center;
  align-items: center;
  background: #f9fafb;
  height: 120px;
  border-radius: 4px;
}
.popup-box-content.text-content {
  justify-content: flex-start;
  align-items: flex-start;
  padding: 12px;
  overflow-y: auto;
  line-height: 1.5;
  font-size: 14px;
}
.placeholder-img {
  height: 80px;
  opacity: 0.5;
}

/* Modal 弹窗样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1100;
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-container {
  width: 700px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  display: flex;
  flex-direction: column;
  animation: fadeIn 0.2s ease-out;
}
@keyframes fadeIn {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}
.modal-header {
  padding: 16px 20px;
  border-bottom: 1px solid var(--line);
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.modal-header h3 {
  margin: 0;
  font-size: 16px;
  color: var(--text-primary);
}
.modal-body {
  padding: 20px;
}
.modal-footer {
  padding: 16px 20px;
  border-top: 1px solid var(--line);
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

/* 穿梭框组件样式 */
.transfer-container {
  display: flex;
  align-items: center;
  gap: 16px;
}
.transfer-panel {
  flex: 1;
  border: 1px solid var(--line);
  border-radius: 4px;
  height: 360px;
  display: flex;
  flex-direction: column;
}
.transfer-panel-header {
  padding: 12px;
  background-color: #f9fafb;
  border-bottom: 1px solid var(--line);
  font-weight: 600;
  font-size: 14px;
}
.text-orange {
  color: #f59e0b; /* 橙色 */
}
.transfer-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px 0;
}
.transfer-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  cursor: pointer;
  font-size: 14px;
  color: var(--text-primary);
}
.transfer-item input[type="checkbox"] {
  width: 16px;
  height: 16px;
  cursor: pointer;
  accent-color: var(--primary); /* 现代浏览器支持的原生 checkbox 颜色修改，与业务审批页面保持视觉一致 */
}
.transfer-item span {
  white-space: nowrap; /* 确保文字一行显示完整 */
  overflow: hidden;
  text-overflow: ellipsis;
}
.transfer-item:hover {
  background-color: #f3f4f6;
}
.transfer-actions {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.transfer-actions .btn {
  width: 100px; /* 加宽以容纳“全部取消授权”等长文案 */
}
</style>