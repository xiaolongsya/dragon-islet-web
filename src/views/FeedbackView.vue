<template>
  <div class="view-feedback">
    <div class="my-msg-header">✉️ 投递鳞笺</div>
    <div class="fb-form">
      <p class="fb-tip">无论是功能构想、存世疑虑，还是对龙屿的期许，皆可在此投递。龙主将亲阅每一封信笺。</p>
      <textarea v-model="localContent" placeholder="在此写下您的信笺..." maxlength="500"></textarea>
      <div class="fb-actions">
        <button class="btn-send-fb" :disabled="!localContent.trim()" @click="handleSubmit">投递信笺</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
const emit = defineEmits(['submit']);
const localContent = ref('');
const handleSubmit = () => {
  emit('submit', localContent.value);
  localContent.value = '';
};
</script>

<style scoped>
.view-feedback { flex: 1; display: flex; flex-direction: column; padding: 40px; max-width: 800px; margin: 0 auto; width: 100%; overflow-y: auto; }
.my-msg-header { font-size: 1.2rem; font-weight: 600; color: #eee; margin-bottom: 20px; border-left: 4px solid #c0392b; padding-left: 12px; letter-spacing: 2px; }
.fb-form { margin-top: 30px; background: rgba(255,255,255,0.02); padding: 30px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.05); }
.fb-tip { color: #666; font-size: .9rem; margin-bottom: 20px; line-height: 1.6; }
.fb-form textarea { width: 100%; height: 200px; background: rgba(0,0,0,0.2); border: 1px solid #333; border-radius: 12px; color: #ccc; padding: 16px; font-size: 1rem; resize: none; outline: none; transition: .3s; }
.fb-form textarea:focus { border-color: #c0392b; }
.fb-actions { display: flex; justify-content: flex-end; margin-top: 20px; }
.btn-send-fb { background: #c0392b; color: #fff; border: none; border-radius: 10px; padding: 12px 32px; font-weight: bold; cursor: pointer; transition: .2s; }
.btn-send-fb:hover:not(:disabled) { background: #e74c3c; transform: translateY(-2px); }
.btn-send-fb:disabled { opacity: 0.3; cursor: not-allowed; }
</style>
