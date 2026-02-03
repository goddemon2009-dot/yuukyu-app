<template>
  <div class="app-container">
    <aside class="sidebar no-print">
      <h2 class="title">有給申請</h2>
      <div class="input-form">
        <div class="field">
          <label>氏名</label>
          <input v-model="form.name" type="text" placeholder="氏名を入力">
        </div>
        <div class="field">
          <label>日付</label>
          <input v-model="form.date" type="date">
        </div>
        <div class="field">
          <label>理由</label>
          <textarea v-model="form.reason" rows="3" placeholder="私用のため等"></textarea>
        </div>
        <button @click="addLog" class="btn-primary">申請を登録</button>
      </div>
    </aside>

    <main class="log-area no-print">
      <h2 class="title">申請履歴・承認管理</h2>
      <div class="log-grid">
        <div v-for="log in logs" :key="log.id" class="log-card">
          <div class="log-info">
            <span class="log-date">{{ log.date }}</span>
            <span class="log-name">{{ log.name }}</span>
            <p class="log-reason">{{ log.reason }}</p>
          </div>
          <div class="log-status">
            <span :class="['badge', log.status]">{{ log.status }}</span>
          </div>
          <div class="log-btns">
            <button v-if="log.status === '申請中'" @click="approve(log.id)" class="btn-approve">承認</button>
            <button @click="openPrint(log)" class="btn-print">印刷用表示</button>
          </div>
        </div>
      </div>
    </main>

    <div v-if="target" class="print-page">
      <div class="a4-sheet">
        <h1>有給休暇申請書</h1>
        <div class="print-body">
          <p>【申請者】 {{ target.name }}</p>
          <p>【休暇日】 {{ target.date }}</p>
          <p>【事由】 {{ target.reason }}</p>
          <p>【状態】 {{ target.status }}</p>
        </div>
        <div class="stamp-box">承認印</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const logs = ref([
  { id: 1, name: 'しょうや', date: '2026-02-04', reason: '役所手続き', status: '承認済み' },
  { id: 2, name: '職員A', date: '2026-02-05', reason: '通院', status: '申請中' }
])

const form = ref({ name: '', date: '', reason: '' })
const target = ref(null)

const addLog = () => {
  if (!form.value.name) return
  logs.value.unshift({ id: Date.now(), ...form.value, status: '申請中' })
  form.value = { name: '', date: '', reason: '' }
}

const approve = (id) => {
  const item = logs.value.find(l => l.id === id)
  if (item) item.status = '承認済み'
}

const openPrint = (item) => {
  target.value = item
  setTimeout(() => {
    window.print()
    target.value = null
  }, 100)
}
</script>

<style scoped>
/* 背景を「白」、文字を「濃いグレー」に強制固定 */
.app-container {
  display: flex;
  min-height: 100vh;
  background-color: #ffffff !important;
  color: #333333 !important;
  font-family: 'Helvetica', 'Arial', sans-serif;
}

.sidebar {
  width: 320px;
  background: #f8f9fa;
  border-right: 1px solid #dee2e6;
  padding: 25px;
}

.log-area {
  flex: 1;
  padding: 25px;
  background: #ffffff;
}

.title { border-bottom: 2px solid #ff4500; padding-bottom: 10px; margin-bottom: 20px; color: #333 !important; }

.field { margin-bottom: 15px; }
.field label { display: block; font-weight: bold; margin-bottom: 5px; }
input, textarea { width: 100%; padding: 10px; border: 1px solid #ced4da; border-radius: 4px; background: white !important; color: black !important; }

.btn-primary { width: 100%; padding: 12px; background: #ff4500; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; }

.log-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  border: 1px solid #dee2e6;
  border-radius: 8px;
  margin-bottom: 10px;
  background: white;
}

.badge { padding: 4px 10px; border-radius: 20px; font-size: 0.8rem; font-weight: bold; }
.badge.申請中 { background: #fff3cd; color: #856404; }
.badge.承認済み { background: #d4edda; color: #155724; }

.btn-approve { background: #28a745; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; margin-right: 5px; }
.btn-print { background: #6c757d; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; }

/* 印刷用 */
@media print {
  .no-print { display: none !important; }
  .print-page { display: block !important; position: fixed; top: 0; left: 0; width: 100%; background: white !important; }
  .a4-sheet { width: 100%; padding: 40px; color: black !important; }
  .stamp-box { border: 2px solid red; color: red; width: 80px; height: 80px; display: flex; align-items: center; justify-content: center; float: right; }
}
@media screen { .print-page { display: none; } }
</style>