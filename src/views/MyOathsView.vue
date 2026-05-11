<template>
  <div class="view-my-oaths layout">
    <section class="chat-area">
      <div class="msg-list">
        <div class="my-oaths-header">
          <h2>我的历史誓言</h2>
          <span class="total-txt">共 {{ total }} 条</span>
        </div>
        <div v-if="items.length===0" class="empty-tip">尚未留下任何誓言</div>
        <transition-group name="t-msg">
          <div v-for="msg in items" :key="msg.ID" class="msg-row" :class="{'msg-recalled': msg.is_recalled}">
             <div v-if="msg.is_recalled" class="recall-note">✦ 你撤回了一条消息 ({{ fmtDate(msg.CreatedAt) }})</div>
             <div v-else class="msg-body my-msg-body">
              <div class="msg-meta">
                <span class="msg-name">{{ user.username }}</span>
                <span class="itag" :class="msg.ai_interest?'itag-fire':'itag-void'">{{ msg.ai_interest?'🔥 青睐':'🌪 不屑' }}</span>
                <span class="msg-time">{{ fmtDate(msg.CreatedAt) }}</span>
              </div>
              <div class="bubble b-user">
                {{ msg.content }}
                <button class="btn-del" @click="$emit('del-msg', msg.ID)" title="抹除誓言"><span class="del-ic">✦</span></button>
              </div>
            </div>
          </div>
        </transition-group>
        <div class="pagination" v-if="total > limit">
          <button :disabled="page===1" @click="$emit('change-page', page-1)">←</button>
          <span>{{ page }} / {{ Math.ceil(total/limit) }}</span>
          <button :disabled="page >= Math.ceil(total/limit)" @click="$emit('change-page', page+1)">→</button>
        </div>

        <div class="m-div" style="margin: 40px 0;"></div>

        <div class="my-msg-header">✉️ 我的鳞笺记录</div>
        <div v-if="feedbacks.length===0" class="empty-tip">尚未向龙主投递过信笺</div>
        <div v-for="fb in feedbacks" :key="fb.ID" class="fb-item">
          <div class="fb-q">
            <span class="fb-ic">问</span> {{ fb.content }}
            <span class="fb-time">{{ fmtDate(fb.CreatedAt) }}</span>
          </div>
          <div class="fb-a">
            <div v-if="fb.is_replied" class="fb-reply">
              <span class="reply-tag">主的回响:</span> {{ fb.reply_content }}
            </div>
            <div v-else class="fb-wait">
              等待龙语回响...
              <button class="btn-fb-del" @click="$emit('del-fb', fb.ID)">抹除</button>
            </div>
          </div>
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
  feedbacks: Array,
  user: Object
});
defineEmits(['change-page', 'del-msg', 'del-fb']);

const fmtDate = t => new Date(t).toLocaleString('zh-CN');
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;}
.msg-list{flex:1;overflow-y:auto;padding:24px 48px;}
.my-oaths-header { display: flex; align-items: baseline; gap: 16px; margin-bottom: 32px; border-bottom: 1px solid rgba(192,57,43,0.2); padding-bottom: 12px; }
.my-oaths-header h2 { font-family: 'Noto Serif SC', serif; font-size: 1.5rem; color: #f0f0f0; letter-spacing: 2px; }
.my-msg-header { font-size: 1.2rem; font-weight: 600; color: #eee; margin-bottom: 20px; border-left: 4px solid #c0392b; padding-left: 12px; letter-spacing: 2px; }
.total-txt { font-size: .8rem; color: #444; }
.my-msg-body { width: 100%; }
.pagination { display: flex; align-items: center; justify-content: center; gap: 20px; margin-top: 40px; }
.pagination button { background: none; border: 1px solid #333; color: #888; width: 32px; height: 32px; border-radius: 6px; cursor: pointer; transition: .2s; }
.pagination button:hover:not(:disabled) { border-color: #c0392b; color: #c0392b; box-shadow: 0 0 12px rgba(192,57,43,0.3); }
.pagination button:disabled { opacity: 0.2; cursor: not-allowed; }
.empty-tip { text-align: center; color: #333; padding: 100px 0; letter-spacing: 4px; }
.m-div { height: 1px; background: rgba(255,255,255,0.05); }

.msg-row{display:flex;gap:14px;margin-bottom:24px;align-items:flex-start;}
.msg-meta{display:flex;align-items:center;gap:8px;margin-bottom:6px;flex-wrap:wrap;}
.msg-name{font-size:.85rem;font-weight:600;color:#bbb;}
.msg-time{font-size:.7rem;color:#444;margin-left:auto;}
.itag{font-size:.7rem;font-weight:600;padding:3px 9px;border-radius:6px;letter-spacing: 1px;}
.itag-fire{background:rgba(192,57,43,.12);color:#e74c3c;border:1px solid rgba(192,57,43,.3);}
.itag-void{background:rgba(255,255,255,.03);color:#666;border:1px solid rgba(255,255,255,.06);}
.bubble{padding:12px 16px;border-radius:4px 14px 14px 14px;line-height:1.75;font-size:.9rem;max-width:600px;position:relative;}
.b-user{background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);color:#ccc;}
.btn-del{position:absolute;right:-28px;top:50%;transform:translateY(-50%);background:none;border:none;color:#333;font-size:1rem;cursor:pointer;opacity:0;transition:.3s;padding:8px;}
.del-ic { font-size: 1.2rem; }
.bubble:hover .btn-del{opacity:1;color:#c0392b;}

.fb-item { background: rgba(255,255,255,0.01); border: 1px solid rgba(255,255,255,0.03); border-radius: 12px; padding: 20px; margin-bottom: 16px; }
.fb-q { font-size: .95rem; color: #ccc; margin-bottom: 12px; display: flex; align-items: flex-start; gap: 10px; }
.fb-ic { background: #333; color: #aaa; font-size: .65rem; padding: 2px 6px; border-radius: 4px; }
.fb-time { font-size: .7rem; color: #333; margin-left: auto; }
.fb-a { border-top: 1px solid rgba(255,255,255,0.03); padding-top: 12px; }
.fb-reply { color: #888; font-size: .9rem; line-height: 1.6; }
.reply-tag { color: #c0392b; font-weight: bold; margin-right: 8px; }
.fb-wait { font-size: .8rem; color: #444; }
.btn-fb-del { margin-left: 12px; background: none; border: 1px solid rgba(192,57,43,0.3); color: #c0392b; font-size: .7rem; padding: 3px 8px; border-radius: 6px; cursor: pointer; transition: .2s; }
.btn-fb-del:hover { background: rgba(192,57,43,0.1); }
</style>
