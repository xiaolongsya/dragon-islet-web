<template>
  <div class="view-admin layout">
    <section class="chat-area">
      <div class="msg-list">
        <div class="admin-header">
          <div class="admin-title-wrap">
            <div class="my-msg-header">🛡️ 龙屿守护者 · 鳞笺审阅</div>
          </div>
          <div class="admin-actions-top">
            <button class="btn-p btn-outline" :disabled="isUpdating" @click="$emit('update-manifesto')">
              <span v-if="isUpdating" class="spin-sm">🌀</span>
              {{ isUpdating ? '正在重塑架构...' : '✦ 更新系统架构' }}
            </button>
            <button class="btn-p btn-glow" :disabled="isGenerating" @click="$emit('manual-generate')">
              <span v-if="isGenerating" class="spin-sm">🌀</span>
              {{ isGenerating ? '正在追溯时空...' : '✦ 手动降下今日史诗' }}
            </button>
          </div>
        </div>
        <div v-if="items.length===0" class="empty-tip">暂无待审阅的鳞笺</div>
        <div v-for="fb in items" :key="fb.id" class="admin-fb-card">
          <div class="fb-card-top">
            <span class="fb-user-id">游侠 ID: {{ fb.user_id }}</span>
            <span class="fb-time">{{ fmtDate(fb.created_at) }}</span>
          </div>
          <div class="fb-card-content">“{{ fb.content }}”</div>
          <div v-if="fb.is_replied" class="fb-replied-box">
            <span class="reply-tag">已回响:</span> {{ fb.reply_content }}
          </div>
          <div v-else class="fb-reply-form">
             <textarea v-model="fb.replyInput" placeholder="在此赐予龙语回响..."></textarea>
             <button class="btn-p btn-sm" @click="$emit('reply', fb)">赐予回响</button>
          </div>
        </div>
        <div class="pagination" v-if="total > limit">
          <button :disabled="page===1" @click="$emit('change-page', page-1)">←</button>
          <span>{{ page }} / {{ Math.ceil(total/limit) }}</span>
          <button :disabled="page >= Math.ceil(total/limit)" @click="$emit('change-page', page+1)">→</button>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
defineProps({
  items: Array,
  total: Number,
  page: Number,
  limit: Number,
  isGenerating: Boolean,
  isUpdating: Boolean
});
defineEmits(['reply', 'change-page', 'manual-generate', 'update-manifesto']);

const fmtDate = t => new Date(t).toLocaleString('zh-CN');
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;background: transparent;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;background: linear-gradient(to right, rgba(0,0,0,0.85) 0%, rgba(0,0,0,0.6) 50%, rgba(0,0,0,0.85) 100%); transform: translateZ(0); will-change: transform;}
.msg-list{flex:1;overflow-y:auto;padding:40px 60px;background: transparent;}
.admin-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 40px; }
.admin-title-wrap { display: flex; align-items: center; gap: 16px; }
.admin-logo { width: 44px; height: 44px; border-radius: 12px; border: 1px solid rgba(192,57,43,0.3); }
.my-msg-header { font-size: 1.5rem; font-weight: 600; color: #eee; border-left: 4px solid #c0392b; padding-left: 16px; font-family: 'Noto Serif SC', serif; letter-spacing: 4px; }
.admin-actions-top { display: flex; gap: 16px; }
.btn-outline { background: none; border: 1px solid rgba(192,57,43,0.5); color: #c0392b; }
.btn-outline:hover:not(:disabled) { background: rgba(192,57,43,0.1); }
.btn-glow { box-shadow: 0 0 15px rgba(192,57,43,0.2); animation: pulse-btn 3s infinite; display: flex; align-items: center; gap: 8px; }
.btn-glow:disabled { opacity: 0.7; cursor: wait; animation: none; }
.spin-sm { display: inline-block; animation: spin 1s linear infinite; font-size: 1rem; }
@keyframes pulse-btn { 0%,100%{box-shadow:0 0 15px rgba(192,57,43,0.2)} 50%{box-shadow:0 0 25px rgba(192,57,43,0.5)} }
@keyframes spin { to { transform: rotate(360deg); } }
.empty-tip { text-align: center; color: #333; padding: 100px 0; letter-spacing: 4px; }
.pagination { display: flex; align-items: center; justify-content: center; gap: 20px; margin-top: 40px; }
.pagination button { background: none; border: 1px solid #333; color: #888; width: 32px; height: 32px; border-radius: 6px; cursor: pointer; transition: .2s; }
.pagination button:hover:not(:disabled) { border-color: #c0392b; color: #c0392b; box-shadow: 0 0 12px rgba(192,57,43,0.3); }
.pagination button:disabled { opacity: 0.2; cursor: not-allowed; }

.admin-fb-card { background: rgba(20,20,20,0.8); border: 1px solid rgba(255,255,255,0.06); border-radius: 20px; padding: 32px; margin-bottom: 32px; transform: translateZ(0); will-change: transform; box-shadow: 0 10px 30px rgba(0,0,0,0.4); }
.fb-card-top { display: flex; justify-content: space-between; font-size: .85rem; color: #555; margin-bottom: 16px; border-bottom: 1px solid rgba(255,255,255,0.04); padding-bottom: 12px; }
.fb-card-content { color: #f0f0f0; font-size: 1.1rem; line-height: 1.8; margin-bottom: 24px; font-style: italic; }
.fb-reply-form textarea { width: 100%; height: 100px; background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.1); border-radius: 12px; color: #eee; padding: 16px; font-size: 1rem; resize: none; margin-bottom: 16px; outline: none; transition: 0.3s; }
.fb-reply-form textarea:focus { border-color: rgba(192,57,43,0.6); box-shadow: 0 0 15px rgba(192,57,43,0.1); }
.fb-replied-box { background: rgba(192,57,43,0.08); padding: 20px; border-radius: 12px; font-size: 1rem; color: #aaa; border: 1px solid rgba(192,57,43,0.2); line-height: 1.6; }
.reply-tag { color: #ff4d4d; font-weight: bold; margin-right: 12px; }
.btn-p { background: #c0392b; color: #fff; border: none; border-radius: 10px; padding: 10px 24px; font-weight: bold; cursor: pointer; transition: .2s; }
.btn-p:hover { background: #e74c3c; }
.btn-sm { padding: 6px 16px; font-size: .8rem; }
</style>
