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
                  <div v-if="msg.content.includes('![](')" class="msg-img-wrap">
                    <img :src="extractImgUrl(msg.content)" class="chat-img" @click="openImg(extractImgUrl(msg.content))">
                  </div>
                  <div v-else>{{ msg.content }}</div>
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
          <div class="magic-options" v-if="localMsg.trim()">
            <span class="opt-label">显像画幅:</span>
            <button v-for="s in ['1:1', '16:9', '9:16', '3:2', '2:3']" 
              :key="s" 
              class="btn-opt" 
              :class="{active: selectedSize === s}"
              @click="selectedSize = s"
            >{{ s }}</button>
          </div>
          <textarea v-model="localMsg" placeholder="写下你的誓言，龙主正在聆听..." :disabled="isChecking" maxlength="500" @keydown.ctrl.enter="handleSend"></textarea>
          <div class="input-foot">
            <span class="cd-txt" v-if="postCooldown>0">冷却 {{ postCooldown }}s</span>
            <span class="hint" v-else>Ctrl+Enter 发送</span>
            <div class="btn-group">
              <button 
                class="btn-magic" 
                @click="handleMagic" 
                :disabled="isChecking || !localMsg.trim()"
                title="龙息幻化"
              >
                <span class="magic-ic">✨</span>
              </button>
              <button class="btn-p" @click="handleSend" :disabled="isChecking||postCooldown>0||!localMsg.trim()">发布誓言</button>
            </div>
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

const emit = defineEmits(['send', 'load-more', 'del-msg', 'force-reply', 'open-modal', 'set-box-ref', 'generate-image']);

const localMsg = ref('');
const selectedSize = ref('1:1');
const msgBox = ref(null);

const handleSend = () => {
  if(!localMsg.value.trim()) return;
  emit('send', localMsg.value);
  localMsg.value = '';
};

const handleMagic = () => {
  if(!localMsg.value.trim()) return;
  emit('generate-image', { prompt: localMsg.value, size: selectedSize.value });
  localMsg.value = '';
};

const fmtTime = t => new Date(t).toLocaleTimeString('zh-CN',{hour:'2-digit',minute:'2-digit'});

const extractImgUrl = (content) => {
  const match = content.match(/!\[\]\((.*?)\)/);
  return match ? match[1] : '';
};

const openImg = (url) => {
  window.open(url, '_blank');
};

onMounted(() => {
  emit('set-box-ref', msgBox.value);
});
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;background: transparent;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;background: linear-gradient(to right, rgba(0,0,0,0.85) 0%, rgba(0,0,0,0.6) 50%, rgba(0,0,0,0.85) 100%); transform: translateZ(0); will-change: transform;}
.msg-list{flex:1;overflow-y:auto;padding:40px 60px;background: transparent; -webkit-overflow-scrolling: touch;}
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
.msg-meta{display:flex;align-items:center;gap:12px;margin-bottom:8px;flex-wrap:wrap;}
.msg-name{font-size:.9rem;font-weight:700;color:#efefef; text-shadow: 0 0 10px rgba(255,255,255,0.1);}
.name-dragon{color:#ff4d4d;text-shadow: 0 0 15px rgba(192,57,43,0.6);}
.msg-time{font-size:.75rem;color:#555;margin-left:auto;}
.itag{font-size:.75rem;font-weight:700;padding:4px 12px;border-radius:8px;letter-spacing: 1px;box-shadow: 0 2px 8px rgba(0,0,0,0.3);}
.itag-fire{background: linear-gradient(135deg, rgba(192,57,43,0.3), rgba(192,57,43,0.1)); color:#ff6b6b; border: 1px solid rgba(192,57,43,0.5);}
.itag-void{background: rgba(255,255,255,0.05); color:#888; border: 1px solid rgba(255,255,255,0.1);}
.bubble{padding:16px 20px;border-radius:6px 20px 20px 20px;line-height:1.8;font-size:1rem;max-width:650px;position:relative; transition: 0.3s; box-shadow: 0 4px 15px rgba(0,0,0,0.4);}
.b-user{background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);color:#e0e0e0; backdrop-filter: blur(2px);}
.b-dragon{background: linear-gradient(135deg, rgba(192,57,43,0.15) 0%, rgba(192,57,43,0.05) 100%); border: 1px solid rgba(192,57,43,0.3); color:#f0f0f0; border-radius: 20px; box-shadow: inset 0 0 20px rgba(192,57,43,0.1);}

.msg-img-wrap { margin-top: 8px; border-radius: 12px; overflow: hidden; border: 1px solid rgba(255,255,255,0.1); cursor: zoom-in; transition: .3s; }
.msg-img-wrap:hover { transform: scale(1.02); border-color: #c0392b; }
.chat-img { width: 100%; max-width: 350px; display: block; object-fit: cover; }

.btn-del{position:absolute;right:-28px;top:50%;transform:translateY(-50%);background:none;border:none;color:#333;font-size:1rem;cursor:pointer;opacity:0;transition:.3s;padding:8px;z-index:10;}
.del-ic { font-size: 1.2rem; filter: drop-shadow(0 0 5px rgba(192,57,43,0.2)); }
.bubble:hover .btn-del{opacity:1;color:#c0392b;}
.force-reply-tip { display: flex; align-items: center; gap: 10px; margin-top: 8px; }
.tip-txt { font-size: .9rem; color: #555; font-style: italic; }
.btn-treasure { background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #c0392b; font-size: .8rem; padding: 3px 10px; border-radius: 4px; cursor: pointer; transition: .2s; font-weight: 600; }
.btn-treasure:hover { background: #c0392b; color: white; box-shadow: 0 0 15px rgba(192,57,43,0.4); }
.treasure-used-tag { margin-top: 6px; font-size: .7rem; color: #c0392b; font-weight: 600; letter-spacing: 1px; opacity: 0.6; }

.input-bar{flex-shrink:0;padding:20px 60px 32px;border-top:1px solid rgba(255,255,255,0.05); background: linear-gradient(to top, rgba(0,0,0,0.8), rgba(0,0,0,0.4)); backdrop-filter: blur(15px); }

.magic-options { display: flex; align-items: center; gap: 8px; margin-bottom: 12px; animation: slide-up 0.3s ease; }
.opt-label { font-size: 0.75rem; color: #555; margin-right: 4px; }
.btn-opt { background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08); color: #888; font-size: 0.7rem; padding: 2px 10px; border-radius: 6px; cursor: pointer; transition: 0.2s; }
.btn-opt:hover { color: #ccc; border-color: #444; }
.btn-opt.active { background: rgba(192,57,43,0.1); border-color: #c0392b; color: #c0392b; }

@keyframes slide-up { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

.input-bar textarea{width:100%;height:110px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.1);border-radius:16px;color:#eee;padding:18px;font-size:1.05rem;resize:none;outline:none;transition:0.4s; box-shadow: inset 0 2px 10px rgba(0,0,0,0.5);}
.input-bar textarea:focus{border-color:rgba(192,57,43,0.6);background:rgba(255,255,255,0.04); box-shadow: 0 0 20px rgba(192,57,43,0.15), inset 0 2px 10px rgba(0,0,0,0.5);}
.input-foot{display:flex;align-items:center;justify-content:flex-end;gap:20px;margin-top:12px;}
.cd-txt{font-size:.8rem;color:#c0392b;font-weight:600;letter-spacing:1px;}
.hint{font-size:.75rem;color:#333;letter-spacing:1px;}
.btn-p { background: #c0392b; color: #fff; border: none; border-radius: 10px; padding: 10px 24px; font-weight: bold; cursor: pointer; transition: .2s; }
.btn-p:hover:not(:disabled) { background: #e74c3c; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(192,57,43,0.3); }
.btn-p:disabled { opacity: 0.3; cursor: not-allowed; }

.btn-group { display: flex; gap: 12px; align-items: center; }
.btn-magic { background: rgba(0,0,0,0.5); border: 1px solid rgba(192,57,43,0.3); color: #fff; border-radius: 10px; width: 44px; height: 44px; display: flex; align-items: center; justify-content: center; cursor: pointer; transition: 0.3s; box-shadow: inset 0 0 10px rgba(192,57,43,0.1); }
.btn-magic:hover:not(:disabled) { border-color: #c0392b; box-shadow: 0 0 20px rgba(192,57,43,0.3); transform: scale(1.05); }
.btn-magic:disabled { opacity: 0.2; cursor: not-allowed; filter: grayscale(1); }
.magic-ic { font-size: 1.2rem; filter: drop-shadow(0 0 5px #c0392b); animation: magic-float 2s ease-in-out infinite; }

@keyframes magic-float {
  0%, 100% { transform: translateY(0) rotate(0); }
  50% { transform: translateY(-3px) rotate(10deg); }
}

.btn-sm { padding: 6px 16px; font-size: .8rem; }

.msg-recalled { justify-content: center; margin: 16px 0; }
.recall-note { font-size: .75rem; color: #444; background: rgba(255,255,255,0.02); padding: 4px 16px; border-radius: 20px; letter-spacing: 1px; }

.t-msg-enter-active { transition: all 0.4s ease; }
.t-msg-enter-from { opacity: 0; transform: translateY(20px); }
</style>
