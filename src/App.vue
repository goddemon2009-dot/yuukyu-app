<template>
  <div class="app-container">
    <div v-if="!isLoggedIn" class="login-overlay">
      <div class="login-card">
        <h1 class="text-black">有給管理システム</h1>
        <div class="login-form">
          <input v-model="email" type="email" placeholder="メールアドレス">
          <input v-model="password" type="password" placeholder="パスワード">
          <button @click="handleLogin" class="btn-login">ログイン</button>
        </div>
        <p class="hint">※adminを含むアドレスで管理者としてログイン</p>
      </div>
    </div>

    <div v-else class="dashboard-layout">
      <header class="navbar">
        <span class="status-text">ログイン中: {{ email }} ({{ userRole === 'admin' ? '管理者' : '職員' }})</span>
        <button @click="handleLogout" class="btn-logout">ログアウト</button>
      </header>

      <div class="main-body">
        <aside v-if="userRole === 'staff'" class="sidebar no-print">
          <h2 class="text-black border-bottom">新規有給申請</h2>
          <div class="form-group">
            <label>休暇希望日</label>
            <input v-model="form.date" type="date">
          </div>
          <div class="form-group">
            <label>理由</label>
            <textarea v-model="form.reason" rows="4"></textarea>
          </div>
          <button @click="submitRequest" class="btn-submit">申請を送信する</button>
        </aside>

        <main class="content-panel">
          <h2 class="text-black border-bottom">
            {{ userRole === 'admin' ? '全職員の申請・承認管理' : 'あなたの申請履歴' }}
          </h2>
          <div class="card-list">
            <div v-for="item in filteredRequests" :key="item.id" class="info-card">
              <div class="card-header">
                <span class="user-name">{{ item.userName }}</span>
                <span class="request-date">{{ item.date }}</span>
              </div>
              <p class="request-reason">{{ item.reason }}</p>
              <div class="card-actions">
                <span :class="['badge', item.status]">{{ item.status }}</span>
                <div class="buttons">
                  <button v-if="userRole === 'admin' && item.status === '申請中'" @click="approve(item.id)" class="btn-approve">承認</button>
                  <button @click="startPrint(item)" class="btn-print">印刷</button>
                </div>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>

    <div v-if="printData" class="print-area">
      <div class="print-sheet">
        <h1>有給休暇申請書</h1>
        <p>氏名: {{ printData.userName }}</p>
        <p>日付: {{ printData.date }}</p>
        <p>理由: {{ printData.reason }}</p>
        <p>状態: {{ printData.status }}</p>
        <div class="seal">承認印</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const isLoggedIn = ref(false)
const userRole = ref('staff')
const email = ref('')
const password = ref('')
const printData = ref(null)

const form = ref({ date: '', reason: '' })
const requests = ref([
  { id: 1, userName: 'しょうや', date: '2026-02-10', reason: '私用のため', status: '承認済み' },
  { id: 2, userName: '職員A', date: '2026-02-12', reason: '通院', status: '申請中' }
])

const handleLogin = () => {
  if (!email.value) return
  isLoggedIn.value = true
  userRole.value = email.value.includes('admin') ? 'admin' : 'staff'
}

const handleLogout = () => {
  isLoggedIn.value = false
  email.value = ''
}

const filteredRequests = computed(() => {
  if (userRole.value === 'admin') return requests.value
  return requests.value.filter(r => r.userName === 'しょうや')
})

const submitRequest = () => {
  if (!form.value.date) return
  requests.value.unshift({ id: Date.now(), userName: 'しょうや', ...form.value, status: '申請中' })
  form.value = { date: '', reason: '' }
}

const approve = (id) => {
  const item = requests.value.find(r => r.id === id)
  if (item) item.status = '承認済み'
}

const startPrint = (item) => {
  printData.value = item
  setTimeout(() => { window.print(); printData.value = null; }, 100)
}
</script>

<style scoped>
/* 背景を「しっかりとした水色」に強制 */
.app-container {
  min-height: 100vh;
  background-color: #b3e5fc !important; /* 明るい水色 */
  color: #000000 !important;
  font-family: sans-serif;
}

.text-black { color: #000000 !important; }
.border-bottom { border-bottom: 2px solid #ff5722; padding-bottom: 10px; margin-bottom: 20px; }

/* ログイン */
.login-overlay { display: flex; justify-content: center; align-items: center; height: 100vh; }
.login-card { background: white; padding: 40px; border-radius: 12px; box-shadow: 0 8px 20px rgba(0,0,0,0.2); width: 360px; text-align: center; }
.login-form input { width: 100%; padding: 12px; margin-bottom: 10px; border: 1px solid #ccc; border-radius: 4px; }
.btn-login { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; }

/* ダッシュボード */
.dashboard-layout { display: flex; flex-direction: column; height: 100vh; }
.navbar { background: #333; color: white; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center; }
.btn-logout { background: transparent; border: 1px solid white; color: white; padding: 5px 10px; cursor: pointer; border-radius: 4px; }

.main-body { display: flex; flex: 1; overflow: hidden; }
.sidebar { width: 300px; background: rgba(255, 255, 255, 0.8); padding: 20px; border-right: 1px solid #90caf9; }
.content-panel { flex: 1; padding: 20px; overflow-y: auto; }

/* フォーム・カード */
.form-group { margin-bottom: 15px; }
label { display: block; font-weight: bold; margin-bottom: 5px; color: #000; }
input, textarea { width: 100%; padding: 10px; border: 1px solid #999; border-radius: 4px; background: white !important; color: black !important; }
.btn-submit { width: 100%; padding: 12px; background: #ff5722; color: white; border: none; border-radius: 6px; cursor: pointer; font-weight: bold; }

.info-card { background: white; border-radius: 8px; padding: 15px; margin-bottom: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.card-header { display: flex; justify-content: space-between; font-weight: bold; color: #000; margin-bottom: 10px; }
.request-reason { background: #f0f0f0; padding: 10px; border-radius: 4px; color: #333; margin-bottom: 10px; }

.card-actions { display: flex; justify-content: space-between; align-items: center; }
.badge { padding: 4px 12px; border-radius: 20px; font-weight: bold; font-size: 0.8rem; }
.badge.申請中 { background: #fff3cd; color: #856404; }
.badge.承認済み { background: #d4edda; color: #155724; }

.btn-approve { background: #28a745; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; margin-right: 5px; }
.btn-print { background: #6c757d; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; }

/* 印刷 */
@media print {
  .no-print { display: none !important; }
  .app-container { background: white !important; }
  .print-area { display: block !important; }
}
@media screen { .print-area { display: none; } }
</style>