<template>
  <div class="view-chat layout">
    <section class="chat-area">
      <div class="msg-list chat-msg-list" ref="msgBox">
        <transition-group name="t-msg" tag="div" class="msg-list-inner">
          <div v-for="msg in messages" :key="msg.ID" class="msg-row" :class="{'msg-recalled': msg.is_recalled}">
            <template v-if="msg.is_recalled">
              <div class="recall-note">✦ {{ msg.user?.username || '游侠' }} 撤回了一条消息</div>
            </template>
            <template v-else>
              <div class="msg-av"><img :src="msg.is_ai_reply?dragonAv:(msg.user?.avatar||defAv)"></div>
              <div class="msg-body">
                <div class="msg-meta">
                  <span class="msg-name" :class="msg.is_ai_reply?'name-dragon':''">{{ msg.is_ai_reply?'龙屿之主':(msg.user?.username||'游侠') }}</span>
                  <span v-if="!msg.is_ai_reply" class="itag" :class="msg.ai_interest?'itag-fire':'itag-void'">{{ msg.ai_interest?'🔥 青睐':'🌪 不屑' }}</span>
                  <span class="msg-time">{{ fmtTime(msg.CreatedAt) }}</span>
                </div>
                <div class="bubble" :class="msg.is_ai_reply?'b-dragon':'b-user'">
                  {{ msg.content }}
                  <button v-if="isLoggedIn && msg.user_id === user.ID" class="btn-del" @click="$emit('del-msg', msg.ID)" title="抹除誓言">
                    <span class="del-ic">✦</span>
                  </button>
                </div>
                <div v-if="isLoggedIn && msg.user_id === user.ID && !msg.ai_interest && !msg.is_ai_reply && !msg.is_force_replied" class="force-reply-tip">
                  <span class="tip-txt">龙主未曾侧目...</span>
                  <button class="btn-treasure" @click="$emit('force-reply', msg.ID)">动用秘宝</button>
                </div>
                <div v-if="msg.is_force_replied" class="treasure-used-tag">✦ 秘宝已启 ✦</div>
              </div>
            </template>
          </div>
        </transition-group>

        <div class="load-more-wrap">
          <button v-if="hasMore" @click="$emit('load-more')" :disabled="loadingMore" class="btn-more">
            {{ loadingMore ? '追溯中...' : '查看更早的誓言' }}
          </button>
          <span v-else-if="messages.length>0" class="no-more">已到最初的誓言</span>
        </div>
      </div>

      <div class="input-bar">
        <div v-if="!isLoggedIn" class="guest-tip" @click="$emit('open-modal', 'login')">
          <span>✦ 唤醒龙魂后方可留下誓言</span>
          <button class="btn-p btn-sm">立即登录</button>
        </div>
        <template v-else>
          <textarea v-model="localMsg" placeholder="写下你的誓言，龙主正在聆听..." :disabled="isChecking" maxlength="500" @keydown.ctrl.enter="handleSend"></textarea>
          <div class="input-foot">
            <span class="cd-txt" v-if="postCooldown>0">冷却 {{ postCooldown }}s</span>
            <span class="hint" v-else>Ctrl+Enter 发送</span>
            <button class="btn-p" @click="handleSend" :disabled="isChecking||postCooldown>0||!localMsg.trim()">发布誓言</button>
          </div>
        </template>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';

const props = defineProps({
  messages: Array,
  hasMore: Boolean,
  loadingMore: Boolean,
  isLoggedIn: Boolean,
  user: Object,
  dragonAv: String,
  defAv: String,
  isChecking: Boolean,
  postCooldown: Number
});

const emit = defineEmits(['send', 'load-more', 'del-msg', 'force-reply', 'open-modal', 'set-box-ref']);

const localMsg = ref('');
const msgBox = ref(null);

const handleSend = () => {
  if(!localMsg.value.trim()) return;
  emit('send', localMsg.value);
  localMsg.value = '';
};

const fmtTime = t => new Date(t).toLocaleTimeString('zh-CN',{hour:'2-digit',minute:'2-digit'});

onMounted(() => {
  emit('set-box-ref', msgBox.value);
});
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;}
.msg-list{flex:1;overflow-y:auto;padding:24px 48px;}
.chat-msg-list { display: flex; flex-direction: column-reverse; }
.msg-list-inner { display: flex; flex-direction: column-reverse; }
.msg-list::-webkit-scrollbar{width:3px;}
.msg-list::-webkit-scrollbar-thumb{background:#1a1a1a;border-radius:3px;}

.load-more-wrap{text-align:center;padding:12px 0 20px;}
.btn-more{background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);color:#666;padding:8px 24px;border-radius:20px;cursor:pointer;font-size:.78rem;transition:.2s;font-family:inherit;}
.btn-more:hover:not(:disabled){background:rgba(255,255,255,0.07);color:#aaa;}
.btn-more:disabled{opacity:.4;cursor:not-allowed;}
.no-more{font-size:.7rem;color:#333;letter-spacing:2px;}

.msg-row{display:flex;gap:14px;margin-bottom:24px;align-items:flex-start;}
.msg-av{width:40px;height:40px;border-radius:10px;overflow:hidden;border:1px solid rgba(255,255,255,0.06);flex-shrink:0;}
.msg-av img{width:40px;height:40px;object-fit:cover;display:block;}
.msg-meta{display:flex;align-items:center;gap:8px;margin-bottom:6px;flex-wrap:wrap;}
.msg-name{font-size:.85rem;font-weight:600;color:#bbb;}
.name-dragon{color:#c0392b;text-shadow: 0 0 10px rgba(192,57,43,0.3);}
.msg-time{font-size:.7rem;color:#444;margin-left:auto;}
.itag{font-size:.7rem;font-weight:600;padding:3px 9px;border-radius:6px;letter-spacing: 1px;}
.itag-fire{background:rgba(192,57,43,.12);color:#e74c3c;border:1px solid rgba(192,57,43,.3);box-shadow: 0 0 10px rgba(192,57,43,0.1);}
.itag-void{background:rgba(255,255,255,.03);color:#666;border:1px solid rgba(255,255,255,.06);}
.bubble{padding:12px 16px;border-radius:4px 14px 14px 14px;line-height:1.75;font-size:.9rem;max-width:600px;position:relative;}
.b-user{background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);color:#ccc;}
.b-dragon{background:rgba(192,57,43,.06);border:1px solid rgba(192,57,43,.15);color:#e5e5e5;border-radius:14px;}
.btn-del{position:absolute;right:-28px;top:50%;transform:translateY(-50%);background:none;border:none;color:#333;font-size:1rem;cursor:pointer;opacity:0;transition:.3s;padding:8px;z-index:10;}
.del-ic { font-size: 1.2rem; filter: drop-shadow(0 0 5px rgba(192,57,43,0.2)); }
.bubble:hover .btn-del{opacity:1;color:#c0392b;}
.force-reply-tip { display: flex; align-items: center; gap: 10px; margin-top: 8px; }
.tip-txt { font-size: .9rem; color: #555; font-style: italic; }
.btn-treasure { background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #c0392b; font-size: .8rem; padding: 3px 10px; border-radius: 4px; cursor: pointer; transition: .2s; font-weight: 600; }
.btn-treasure:hover { background: #c0392b; color: white; box-shadow: 0 0 15px rgba(192,57,43,0.4); }
.treasure-used-tag { margin-top: 6px; font-size: .7rem; color: #c0392b; font-weight: 600; letter-spacing: 1px; opacity: 0.6; }

.input-bar{flex-shrink:0;padding:16px 48px 24px;border-top:1px solid rgba(255,255,255,0.05);}
.guest-tip{display:flex;align-items:center;justify-content:center;gap:20px;padding:16px;background:rgba(255,255,255,.02);border:1px solid rgba(255,255,255,.06);border-radius:14px;color:#555;font-size:.85rem;cursor:pointer;}
.input-bar textarea{width:100%;height:100px;background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.08);border-radius:14px;color:#ccc;padding:16px;font-size:.95rem;resize:none;outline:none;transition:.3s;}
.input-bar textarea:focus{border-color:rgba(192,57,43,0.4);background:rgba(255,255,255,0.05);}
.input-foot{display:flex;align-items:center;justify-content:flex-end;gap:20px;margin-top:12px;}
.cd-txt{font-size:.8rem;color:#c0392b;font-weight:600;letter-spacing:1px;}
.hint{font-size:.75rem;color:#333;letter-spacing:1px;}
.btn-p { background: #c0392b; color: #fff; border: none; border-radius: 10px; padding: 10px 24px; font-weight: bold; cursor: pointer; transition: .2s; }
.btn-p:hover:not(:disabled) { background: #e74c3c; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(192,57,43,0.3); }
.btn-p:disabled { opacity: 0.3; cursor: not-allowed; }
.btn-sm { padding: 6px 16px; font-size: .8rem; }

.msg-recalled { justify-content: center; margin: 16px 0; }
.recall-note { font-size: .75rem; color: #444; background: rgba(255,255,255,0.02); padding: 4px 16px; border-radius: 20px; letter-spacing: 1px; }

.t-msg-enter-active { transition: all 0.4s ease; }
.t-msg-enter-from { opacity: 0; transform: translateY(20px); }
</style>
