<template>
  <div class="system-root">
    <div v-if="!isLoggedIn" class="login-screen">
      <div class="login-card">
        <h1 class="text-black">有給管理システム</h1>
        <div class="login-form">
          <p class="label-black">メールアドレス</p>
          <input v-model="email" type="email" placeholder="example@mail.com">
          <p class="label-black">パスワード</p>
          <input v-model="password" type="password" placeholder="password">
          <button @click="handleLogin" class="btn-login">ログイン</button>
        </div>
        <p class="demo-hint">※ adminを含むアドレスで管理者、それ以外で職員としてログイン</p>
      </div>
    </div>

    <div v-else class="main-dashboard">
      <header class="navbar">
        <span class="user-status">👤 {{ email }} ({{ userRole === 'admin' ? '管理者' : '職員' }})</span>
        <button @click="handleLogout" class="btn-logout">ログアウト</button>
      </header>

      <div class="layout-body">
        <aside v-if="userRole === 'staff'" class="staff-sidebar no-print">
          <h2 class="text-black title-border">新規申請を行う</h2>
          <div class="white-card">
            <div class="input-group">
              <label>休暇希望日</label>
              <input v-model="form.date" type="date">
            </div>
            <div class="input-group">
              <label>理由</label>
              <textarea v-model="form.reason" rows="4" placeholder="私用、通院など"></textarea>
            </div>
            <button @click="submitRequest" class="btn-orange">申請を送信</button>
          </div>
        </aside>

        <main class="history-section">
          <h2 class="text-black title-border">
            {{ userRole === 'admin' ? '全職員の申請・承認管理' : 'あなたの申請履歴' }}
          </h2>
          <div class="card-container">
            <div v-for="log in filteredLogs" :key="log.id" class="history-card">
              <div class="card-header">
                <span class="user-name">{{ log.userName }}</span>
                <span class="req-date">{{ log.date }}</span>
              </div>
              <div class="card-content">
                <p class="reason-box">{{ log.reason }}</p>
              </div>
              <div class="card-footer">
                <span :class="['status-badge', log.status]">{{ log.status }}</span>
                <div class="btns">
                  <button v-if="userRole === 'admin' && log.status === '申請中'" @click="approve(log.id)" class="btn-approve">承認</button>
                  <button @click="openPrint(log)" class="btn-print">印刷用</button>
                </div>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>

    <div v-if="printItem" class="print-only">
      <div class="a4-sheet">
        <h1>有給休暇申請書</h1>
        <p>氏名: {{ printItem.userName }}</p>
        <p>日付: {{ printItem.date }}</p>
        <p>理由: {{ printItem.reason }}</p>
        <p>状態: {{ printItem.status }}</p>
        <div class="seal-area">承認印</div>
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
const printItem = ref(null)

const form = ref({ date: '', reason: '' })
const allLogs = ref([
  { id: 1, userName: 'しょうや', date: '2026-02-10', reason: '役所の手続き', status: '承認済み' },
  { id: 2, userName: '職員A', date: '2026-02-12', reason: '家族の通院付き添い', status: '申請中' }
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

const filteredLogs = computed(() => {
  if (userRole.value === 'admin') return allLogs.value
  return allLogs.value.filter(l => l.userName === 'しょうや')
})

const submitRequest = () => {
  if (!form.value.date) return
  allLogs.value.unshift({ id: Date.now(), userName: 'しょうや', ...form.value, status: '申請中' })
  form.value = { date: '', reason: '' }
}

const approve = (id) => {
  const target = allLogs.value.find(l => l.id === id)
  if (target) target.status = '承認済み'
}

const openPrint = (log) => {
  printItem.value = log
  setTimeout(() => { window.print(); printItem.value = null; }, 100)
}
</script>

<style>
/* 💡 ここがポイント：HTML全体を水色に固定 */
html, body {
  margin: 0;
  padding: 0;
  background-color: #b3e5fc !important; /* 強力な水色 */
  height: 100%;
}
</style>

<style scoped>
.system-root { min-height: 100vh; font-family: sans-serif; color: #000; }
.text-black { color: #000 !important; }
.label-black { color: #000 !important; font-weight: bold; margin-bottom: 5px; text-align: left; width: 100%; }

/* ログイン画面 */
.login-screen { display: flex; justify-content: center; align-items: center; height: 100vh; }
.login-card { background: white; padding: 40px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.2); width: 380px; text-align: center; }
.login-form { display: flex; flex-direction: column; align-items: center; margin-top: 20px; }
.login-form input { width: 100%; padding: 12px; margin-bottom: 15px; border: 1px solid #ccc; border-radius: 5px; background: white; color: black; }
.btn-login { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; }
.demo-hint { font-size: 0.75rem; color: #555; margin-top: 15px; }

/* ダッシュボード */
.dashboard-layout { display: flex; flex-direction: column; }
.navbar { background: #2c3e50; color: white; padding: 12px 25px; display: flex; justify-content: space-between; align-items: center; }
.btn-logout { background: transparent; border: 1px solid white; color: white; padding: 5px 12px; border-radius: 4px; cursor: pointer; }

.layout-body { display: flex; padding: 20px; gap: 20px; }
.staff-sidebar { width: 320px; }
.history-section { flex: 1; }

.title-border { border-bottom: 3px solid #ff5722; padding-bottom: 10px; margin-bottom: 20px; }

/* カードデザイン */
.white-card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
.input-group { margin-bottom: 15px; }
.input-group label { display: block; margin-bottom: 5px; font-weight: bold; color: #000; }
.input-group input, .input-group textarea { width: 100%; padding: 10px; border: 1px solid #999; border-radius: 5px; background: #fff; color: #000; }

.btn-orange { width: 100%; padding: 12px; background: #ff5722; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; font-size: 1rem; }

.history-card { background: white; border-radius: 12px; padding: 20px; margin-bottom: 15px; box-shadow: 0 6px 12px rgba(0,0,0,0.1); border-left: 6px solid #007bff; }
.card-header { display: flex; justify-content: space-between; font-weight: bold; color: #000; margin-bottom: 10px; }
.reason-box { background: #f8f9fa; padding: 15px; border-radius: 6px; color: #333; margin: 10px 0; }

.card-footer { display: flex; justify-content: space-between; align-items: center; margin-top: 10px; }
.status-badge { padding: 5px 15px; border-radius: 20px; font-weight: bold; font-size: 0.85rem; }
.status-badge.申請中 { background: #fff3cd; color: #856404; }
.status-badge.承認済み { background: #d4edda; color: #155724; }

.btn-approve { background: #28a745; color: white; border: none; padding: 8px 16px; border-radius: 5px; cursor: pointer; margin-right: 8px; font-weight: bold; }
.btn-print { background: #6c757d; color: white; border: none; padding: 8px 16px; border-radius: 5px; cursor: pointer; }

/* 印刷用表示 */
@media print {
  .no-print { display: none !important; }
  body { background: white !important; }
  .print-only { display: block !important; padding: 50px; }
}
@media screen { .print-only { display: none; } }
</style>