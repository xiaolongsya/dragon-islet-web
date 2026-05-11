<template>
  <div class="view-admin layout">
    <section class="chat-area">
      <div class="msg-list">
        <div class="my-msg-header">🛡️ 龙屿守护者 · 鳞笺审阅</div>
        <div v-if="items.length===0" class="empty-tip">暂无待审阅的鳞笺</div>
        <div v-for="fb in items" :key="fb.ID" class="admin-fb-card">
          <div class="fb-card-top">
            <span class="fb-user-id">游侠 ID: {{ fb.user_id }}</span>
            <span class="fb-time">{{ fmtDate(fb.CreatedAt) }}</span>
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
  limit: Number
});
defineEmits(['reply', 'change-page']);

const fmtDate = t => new Date(t).toLocaleString('zh-CN');
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;}
.msg-list{flex:1;overflow-y:auto;padding:24px 48px;}
.my-msg-header { font-size: 1.2rem; font-weight: 600; color: #eee; margin-bottom: 20px; border-left: 4px solid #c0392b; padding-left: 12px; letter-spacing: 2px; }
.empty-tip { text-align: center; color: #333; padding: 100px 0; letter-spacing: 4px; }
.pagination { display: flex; align-items: center; justify-content: center; gap: 20px; margin-top: 40px; }
.pagination button { background: none; border: 1px solid #333; color: #888; width: 32px; height: 32px; border-radius: 6px; cursor: pointer; transition: .2s; }
.pagination button:hover:not(:disabled) { border-color: #c0392b; color: #c0392b; box-shadow: 0 0 12px rgba(192,57,43,0.3); }
.pagination button:disabled { opacity: 0.2; cursor: not-allowed; }

.admin-fb-card { background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.05); border-radius: 16px; padding: 24px; margin-bottom: 24px; }
.fb-card-top { display: flex; justify-content: space-between; font-size: .75rem; color: #444; margin-bottom: 12px; }
.fb-card-content { color: #ccc; font-size: 1rem; line-height: 1.6; margin-bottom: 16px; }
.fb-reply-form textarea { width: 100%; height: 80px; background: rgba(0,0,0,0.3); border: 1px solid #333; border-radius: 8px; color: #aaa; padding: 12px; font-size: .9rem; resize: none; margin-bottom: 10px; outline: none; }
.fb-reply-form textarea:focus { border-color: #c0392b; }
.fb-replied-box { background: rgba(192,57,43,0.05); padding: 12px; border-radius: 8px; font-size: .9rem; color: #888; border: 1px solid rgba(192,57,43,0.1); }
.reply-tag { color: #c0392b; font-weight: bold; margin-right: 8px; }
.btn-p { background: #c0392b; color: #fff; border: none; border-radius: 10px; padding: 10px 24px; font-weight: bold; cursor: pointer; transition: .2s; }
.btn-p:hover { background: #e74c3c; }
.btn-sm { padding: 6px 16px; font-size: .8rem; }
</style>
