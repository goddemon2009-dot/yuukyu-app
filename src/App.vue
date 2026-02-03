<template>
  <div class="container">
    <div class="editor-section no-print">
      <h2>有給休暇申請 作成</h2>
      <div class="form-group">
        <label>氏名:</label>
        <input v-model="form.name" type="text" placeholder="氏名を入力">
      </div>
      <div class="form-group">
        <label>休暇日:</label>
        <input v-model="form.date" type="date">
      </div>
      <div class="form-group">
        <label>理由:</label>
        <textarea v-model="form.reason" placeholder="私用のため、など"></textarea>
      </div>
      <button @click="printPaper" class="print-btn">PDF保存 / 印刷</button>
    </div>

    <div class="paper-section">
      <div class="sheet">
        <h1>有給休暇申請書</h1>
        <div class="content">
          <p>提出日: {{ today }}</p>
          <div class="field-row">
            <span class="label">氏名:</span>
            <span class="value">{{ form.name || '（未入力）' }}</span>
          </div>
          <div class="field-row">
            <span class="label">休暇希望日:</span>
            <span class="value">{{ form.date || '（未入力）' }}</span>
          </div>
          <div class="field-row">
            <span class="label">事由:</span>
            <span class="value">{{ form.reason || '（未入力）' }}</span>
          </div>
        </div>
        <div class="stamp-area">
          <div class="stamp">承認印</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const today = new Date().toLocaleDateString();
const form = ref({
  name: '',
  date: '',
  reason: ''
});

const printPaper = () => {
  window.print();
};
</script>

<style scoped>
/* 画面表示用のスタイル */
.container { display: flex; gap: 40px; padding: 20px; font-family: sans-serif; }
.editor-section { width: 300px; padding: 20px; background: #f4f4f4; border-radius: 8px; }
.form-group { margin-bottom: 15px; }
.form-group label { display: block; margin-bottom: 5px; }
.form-group input, .form-group textarea { width: 100%; padding: 8px; border: 1px solid #ccc; }
.print-btn { width: 100%; padding: 10px; background: #ff4500; color: white; border: none; cursor: pointer; font-weight: bold; }

/* 用紙のスタイル */
.paper-section { flex-grow: 1; display: flex; justify-content: center; }
.sheet { width: 210mm; height: 297mm; padding: 20mm; border: 1px solid #ddd; background: white; box-shadow: 0 0 10px rgba(0,0,0,0.1); position: relative; }
h1 { text-align: center; text-decoration: underline; margin-bottom: 50px; }
.field-row { margin-bottom: 30px; border-bottom: 1px solid #000; padding-bottom: 5px; font-size: 1.2rem; }
.label { font-weight: bold; margin-right: 20px; }
.stamp-area { position: absolute; bottom: 50px; right: 50px; }
.stamp { width: 80px; height: 80px; border: 2px solid #cc0000; color: #cc0000; display: flex; align-items: center; justify-content: center; border-radius: 50%; opacity: 0.5; }

/* 印刷時の設定 */
@media print {
  .no-print { display: none; }
  .container { padding: 0; }
  .sheet { border: none; box-shadow: none; width: 100%; height: auto; }
}
</style>