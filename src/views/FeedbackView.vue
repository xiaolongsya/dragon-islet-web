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
.view-feedback { flex: 1; display: flex; flex-direction: column; padding: 60px 40px; max-width: 900px; margin: 0 auto; width: 100%; overflow-y: auto; background: transparent;}
.my-msg-header { font-size: 1.5rem; font-weight: 600; color: #eee; margin-bottom: 40px; border-left: 4px solid #c0392b; padding-left: 16px; font-family: 'Noto Serif SC', serif; letter-spacing: 4px; }
.fb-form { margin-top: 30px; background: rgba(255,255,255,0.03); padding: 50px; border-radius: 30px; border: 1px solid rgba(255,255,255,0.08); backdrop-filter: blur(10px); box-shadow: 0 20px 50px rgba(0,0,0,0.5); }
.fb-tip { color: #888; font-size: 1.1rem; margin-bottom: 40px; line-height: 2; font-style: italic; }
.fb-form textarea { width: 100%; height: 220px; background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.1); border-radius: 16px; color: #eee; padding: 24px; font-size: 1.1rem; resize: none; outline: none; transition: 0.4s; box-shadow: inset 0 2px 10px rgba(0,0,0,0.5); }
.fb-form textarea:focus { border-color: rgba(192,57,43,0.6); box-shadow: 0 0 20px rgba(192,57,43,0.1), inset 0 2px 10px rgba(0,0,0,0.5); }
.fb-actions { display: flex; justify-content: flex-end; margin-top: 30px; }
.btn-send-fb { background: #c0392b; color: #fff; border: none; border-radius: 12px; padding: 14px 48px; font-weight: bold; cursor: pointer; transition: 0.3s; font-size: 1.05rem; letter-spacing: 2px; }
.btn-send-fb:hover:not(:disabled) { background: #ff4d4d; transform: translateY(-3px); box-shadow: 0 5px 20px rgba(192,57,43,0.4); }
.btn-send-fb:disabled { opacity: 0.3; cursor: not-allowed; }
</style>
