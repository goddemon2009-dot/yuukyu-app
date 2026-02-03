<template>
  <div class="system-container">
    <div v-if="!user" class="login-box">
      <h1>有給管理システム</h1>
      <input v-model="loginEmail" type="email" placeholder="メールアドレス">
      <input v-model="loginPw" type="password" placeholder="パスワード">
      <button @click="handleLogin" class="btn-login">ログイン</button>
    </div>

    <div v-else class="dashboard">
      <header class="top-bar">
        <span>ログイン中: {{ user.email }} ({{ role }})</span>
        <button @click="handleLogout" class="btn-logout">ログアウト</button>
      </header>

      <div class="content-wrapper">
        <aside v-if="role === 'staff'" class="sidebar">
          <h2 class="section-title">新規申請</h2>
          <div class="form-group">
            <label>休暇日</label>
            <input v-model="form.date" type="date">
          </div>
          <div class="form-group">
            <label>理由</label>
            <textarea v-model="form.reason"></textarea>
          </div>
          <button @click="submitRequest" class="btn-primary">申請を送信</button>
        </aside>

        <main class="main-panel">
          <h2 class="section-title">{{ role === 'admin' ? '全職員の申請管理' : '自分の申請履歴' }}</h2>
          <div class="list">
            <div v-for="req in filteredLogs" :key="req.id" class="card">
              <div class="info">
                <p><strong>{{ req.userName }}</strong> ({{ req.date }})</p>
                <p class="reason">{{ req.reason }}</p>
              </div>
              <div class="actions">
                <span :class="['status', req.status]">{{ req.status }}</span>
                <button v-if="role === 'admin' && req.status === '申請中'" @click="approve(req.id)" class="btn-approve">承認</button>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// ユーザー状態（本来はFirebaseから取得）
const user = ref(null) 
const role = ref('staff') // 'admin' または 'staff'
const loginEmail = ref('')
const loginPw = ref('')

const form = ref({ date: '', reason: '' })
const allLogs = ref([
  { id: 1, userId: 'staff01', userName: 'しょうや', date: '2026-02-10', reason: '私用', status: '承認済み' },
  { id: 2, userId: 'staff02', userName: '職員A', date: '2026-02-12', reason: '通院', status: '申請中' }
])

// 役割に応じて履歴をフィルタリング
const filteredLogs = computed(() => {
  if (role.value === 'admin') return allLogs.value
  return allLogs.value.filter(l => l.userId === 'staff01') // 本来は自分のIDで絞り込む
})

// 簡易ログイン処理（明日ここをFirebaseに繋ぎます）
const handleLogin = () => {
  if (loginEmail.value.includes('admin')) {
    role.value = 'admin'; user.value = { email: loginEmail.value }
  } else {
    role.value = 'staff'; user.value = { email: loginEmail.value }
  }
}
const handleLogout = () => user.value = null
const approve = (id) => {
  const item = allLogs.value.find(l => l.id === id)
  if (item) item.status = '承認済み'
}
</script>

<style scoped>
/* 背景と同化しないための強力な設定 */
.system-container { min-height: 100vh; background: #f0f2f5 !important; color: #000000 !important; }
.login-box { max-width: 400px; margin: 100px auto; padding: 40px; background: white; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); text-align: center; }
.login-box input { width: 100%; margin-bottom: 15px; padding: 12px; border: 1px solid #ddd; }

.dashboard { display: flex; flex-direction: column; height: 100vh; }
.top-bar { background: #333; color: white; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center; }

.content-wrapper { display: flex; flex: 1; overflow: hidden; }
.sidebar { width: 320px; background: #fff; padding: 25px; border-right: 1px solid #ddd; }
.main-panel { flex: 1; padding: 25px; overflow-y: auto; background: #f9f9f9; }

/* ラベルを真っ黒に */
label { display: block; font-weight: bold; color: #000 !important; margin-bottom: 8px; }
.section-title { color: #000 !important; border-bottom: 3px solid #ff4500; padding-bottom: 8px; margin-bottom: 20px; }

.card { background: white; padding: 20px; border-radius: 8px; margin-bottom: 12px; display: flex; justify-content: space-between; border: 1px solid #eee; }
.status.申請中 { color: #856404; background: #fff3cd; padding: 4px 10px; border-radius: 20px; }
.status.承認済み { color: #155724; background: #d4edda; padding: 4px 10px; border-radius: 20px; }

.btn-primary { width: 100%; background: #ff4500; color: white; border: none; padding: 12px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.btn-login { width: 100%; background: #007bff; color: white; border: none; padding: 12px; border-radius: 6px; cursor: pointer; }
.btn-approve { background: #28a745; color: white; border: none; padding: 8px 15px; border-radius: 4px; cursor: pointer; margin-left: 10px; }
</style>