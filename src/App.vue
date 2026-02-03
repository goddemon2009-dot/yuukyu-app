<template>
  <div class="app-wrapper">
    <div class="sidebar no-print">
      <h2>有給休暇申請</h2>
      <div class="form-card">
        <label>氏名</label>
        <input v-model="newRequest.name" type="text">
        
        <label>休暇日</label>
        <input v-model="newRequest.date" type="date">
        
        <label>理由</label>
        <textarea v-model="newRequest.reason"></textarea>
        
        <button @click="submitRequest" class="submit-btn">申請を送信</button>
      </div>
    </div>

    <div class="main-content no-print">
      <h2>申請履歴一覧</h2>
      <div class="history-list">
        <div v-for="item in history" :key="item.id" class="history-card">
          <div class="card-info">
            <strong>{{ item.date }}</strong> - {{ item.name }}
            <p class="reason-text">{{ item.reason }}</p>
          </div>
          <div class="card-actions">
            <span :class="['status-badge', item.status]">{{ item.status }}</span>
            <button @click="printThis(item)" class="mini-btn">印刷用表示</button>
            <button v-if="item.status === '承認待ち'" @click="approve(item)" class="approve-btn">承認する</button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="printTarget" class="print-only">
      <div class="print-sheet">
        <h1>有給休暇申請書</h1>
        <div class="print-content">
          <p>氏名: {{ printTarget.name }}</p>
          <p>日付: {{ printTarget.date }}</p>
          <p>理由: {{ printTarget.reason }}</p>
          <p>状態: {{ printTarget.status }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const history = ref([
  { id: 1, name: 'しょうや', date: '2026-02-10', reason: '私用のため', status: '承認待ち' },
  { id: 2, name: 'スタッフA', date: '2026-02-12', reason: '通院', status: '承認済み' }
]);

const newRequest = ref({ name: '', date: '', reason: '' });
const printTarget = ref(null);

const submitRequest = () => {
  if (!newRequest.value.name) return;
  history.value.unshift({
    id: Date.now(),
    ...newRequest.value,
    status: '承認待ち'
  });
  newRequest.value = { name: '', date: '', reason: '' };
};

const approve = (item) => { item.status = '承認済み'; };

const printThis = (item) => {
  printTarget.value = item;
  setTimeout(() => {
    window.print();
    printTarget.value = null;
  }, 100);
};
</script>

<style scoped>
/* 全体の背景を白、文字を黒に固定して同化を防ぐ */
.app-wrapper {
  display: flex;
  min-height: 100vh;
  background-color: #f0f2f5;
  color: #333;
  font-family: sans-serif;
}

.sidebar { width: 300px; padding: 20px; background: #fff; border-right: 1px solid #ddd; }
.main-content { flex: 1; padding: 20px; overflow-y: auto; }

.form-card { display: flex; flex-direction: column; gap: 10px; }
input, textarea { padding: 10px; border: 1px solid #ccc; border-radius: 4px; }

.submit-btn { padding: 12px; background: #ff4500; color: white; border: none; border-radius: 4px; cursor: pointer; }

.history-card {
  background: white;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}

.status-badge { padding: 4px 8px; border-radius: 12px; font-size: 0.8rem; }
.status-badge.承認待ち { background: #fff3cd; color: #856404; }
.status-badge.承認済み { background: #d4edda; color: #155724; }

.approve-btn { background: #28a745; color: white; border: none; padding: 5px 10px; cursor: pointer; border-radius: 4px; margin-left: 10px; }
.mini-btn { background: #eee; border: 1px solid #ccc; padding: 5px 10px; cursor: pointer; border-radius: 4px; }

/* 印刷用スタイル（背景を白、文字を黒に強制） */
@media print {
  .no-print { display: none !important; }
  .print-only { display: block !important; background: white !important; color: black !important; width: 100%; height: 100%; }
  .print-sheet { padding: 40px; border: 1px solid #000; background: white !important; color: black !important; }
}
@media screen { .print-only { display: none; } }
</style>