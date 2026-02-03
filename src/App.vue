<template>
  <div class="app-root">
    <div v-if="!isLoggedIn" class="login-container">
      <div class="login-card">
        <h1 class="black-text">有給管理システム</h1>
        <p class="sub-text">職員・管理者ログイン</p>
        <div class="login-form">
          <input v-model="email" type="email" placeholder="メールアドレス">
          <input v-model="password" type="password" placeholder="パスワード">
          <button @click="login" class="login-btn">ログイン</button>
        </div>
        <p class="hint">※ admin を含むアドレスで「管理者」、それ以外で「職員」としてログイン（デモ）</p>
      </div>
    </div>

    <div v-else class="dashboard">
      <header class="header">
        <span class="user-info">{{ userRole === 'admin' ? '【管理者】' : '【職員】' }} {{ email }}</span>
        <button @click="logout" class="logout-link">ログアウト</button>
      </header>

      <div class="main-layout">
        <aside v-if="userRole === 'staff'" class="sidebar no-print">
          <h2 class="black-text section-title">有給休暇を申請</h2>
          <div class="input-card">
            <div class="form-item">
              <label>休暇日</label>
              <input v-model="form.date" type="date">
            </div>
            <div class="form-item">
              <label>理由</label>
              <textarea v-model="form.reason" rows="4" placeholder="私用のため等"></textarea>
            </div>
            <button @click="addRequest" class="orange-btn">申請を送信</button>
          </div>
        </aside>

        <main class="history-panel">
          <h2 class="black-text section-title">{{ userRole === 'admin' ? '全職員の申請・承認管理' : 'あなたの申請履歴' }}</h2>
          <div class="history-list">
            <div v-for="item in filteredRequests" :key="item.id" class="card">
              <div class="card-body">
                <div class="req-header">
                  <span class="req-user">{{ item.userName }}</span>
                  <span class="req-date">{{ item.date }}</span>
                </div>
                <p class="req-reason">{{ item.reason }}</p>
              </div>
              <div class="card-footer">
                <span :class="['status-badge', item.status]">{{ item.status }}</span>
                <div class="action-btns">
                  <button v-if="userRole === 'admin' && item.status === '申請中'" @click="approve(item.id)" class="approve-btn">承認する</button>
                  <button @click="printDoc(item)" class="print-btn">印刷用表示</button>
                </div>
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>

    <div v-if="printTarget" class="print-preview">
      <div class="a4-page">
        <h1>有給休暇申請書</h1>
        <div class="doc-line">氏名: {{ printTarget.userName }}</div>
        <div class="doc-line">日付: {{ printTarget.date }}</div>
        <div class="doc-line">事由: {{ printTarget.reason }}</div>
        <div class="doc-line">状態: {{ printTarget.status }}</div>
        <div class="stamp">承認印</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// 状態管理
const isLoggedIn = ref(false)
const userRole = ref('staff') // 'admin' or 'staff'
const email = ref('')
const password = ref('')
const printTarget = ref(null)

const form = ref({ date: '', reason: '' })

// 仮のデータ（本来はFirestoreから取得）
const requests = ref([
  { id: 1, userId: 'user1', userName: 'しょうや', date: '2026-02-04', reason: '役所の手続き', status: '承認済み' },
  { id: 2, userId: 'user2', userName: '職員A', date: '2026-02-05', reason: '法事のため', status: '申請中' }
])

// ログイン処理（デモ用）
const login = () => {
  if (!email.value) return
  isLoggedIn.value = true
  userRole.value = email.value.includes('admin') ? 'admin' : 'staff'
}

const logout = () => {
  isLoggedIn.value = false
  email.value = ''
  password.value = ''
}

// 履歴のフィルタリング（職員なら自分のだけ、管理者なら全部）
const filteredRequests = computed(() => {
  if (userRole.value === 'admin') return requests.value
  return requests.value.filter(r => r.userName === 'しょうや') // 実際はログインIDで絞り込む
})

const addRequest = () => {
  if (!form.value.date) return
  requests.value.unshift({
    id: Date.now(),
    userId: 'user1',
    userName: 'しょうや',
    ...form.value,
    status: '申請中'
  })
  form.value = { date: '', reason: '' }
}

const approve = (id) => {
  const item = requests.value.find(r => r.id === id)
  if (item) item.status = '承認済み'
}

const printDoc = (item) => {
  printTarget.value = item
  setTimeout(() => {
    window.print()
    printTarget.value = null
  }, 100)
}
</script>

<style scoped>
/* 全体の水色背景と文字色の固定 */
.app-root {
  min-height: 100vh;
  background-color: #e0f7ff !important; /* 水色 */
  color: #000000 !important;
  font-family: sans-serif;
}

.black-text { color: #000000 !important; }

/* ログイン画面 */
.login-container { display: flex; justify-content: center; align-items: center; height: 100vh; }
.login-card { background: white; padding: 40px; border-radius: 15px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); text-align: center; width: 350px; }
.login-form input { width: 100%; margin-bottom: 15px; padding: 12px; border: 1px solid #ccc; border-radius: 5px; }
.login-btn { width: 100%; background: #007bff; color: white; padding: 12px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; }
.hint { font-size: 0.7rem; color: #666; margin-top: 15px; }

/* ダッシュボード */
.dashboard { display: flex; flex-direction: column; height: 100vh; }
.header { background: #333; color: white; padding: 10px 20px; display: flex; justify-content: space-between; }
.logout-link { background: none; border: 1px solid white; color: white; padding: 2px 10px; cursor: pointer; border-radius: 4px; }

.main-layout { display: flex; flex: 1; overflow: hidden; }
.sidebar { width: 320px; background: #ffffffcc; padding: 20px; border-right: 1px solid #b3e5fc; }
.history-panel { flex: 1; padding: 20px; overflow-y: auto; }

.section-title { border-bottom: 2px solid #ff5722; padding-bottom: 8px; margin-bottom: 20px; }

/* 入力・カードデザイン */
.form-item { margin-bottom: 15px; }
label { display: block; font-weight: bold; margin-bottom: 5px; color: #000; }
input, textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 4px; background: white !important; color: black !important; }

.orange-btn { width: 100%; background: #ff5722; color: white; padding: 12px; border: none; border-radius: 6px; cursor: pointer; font-weight: bold; }

.card { background: white; border-radius: 10px; padding: 15px; margin-bottom: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); display: flex; flex-direction: column; }
.req-header { display: flex; justify-content: space-between; font-weight: bold; margin-bottom: 10px; color: #000; }
.req-reason { background: #f9f9f9; padding: 10px; border-radius: 4px; margin-bottom: 15px; }

.card-footer { display: flex; justify-content: space-between; align-items: center; }
.status-badge { padding: 4px 12px; border-radius: 20px; font-size: 0.8rem; font-weight: bold; }
.status-badge.申請中 { background: #fff3cd; color: #856404; }
.status-badge.承認済み { background: #d4edda; color: #155724; }

.approve-btn { background: #28a745; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; margin-right: 5px; }
.print-btn { background: #6c757d; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; }

/* 印刷用 */
@media print {
  .no-print { display: none !important; }
  .app-root { background: white !important; }
  .print-preview { display: block !important; }
  .a4-page { padding: 50px; color: black !important; }
}
@media screen { .print-preview { display: none; } }
</style>