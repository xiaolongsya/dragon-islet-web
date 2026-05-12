<template>
  <div class="view-chat layout">
    <section class="chat-area" :class="{'is-mobile': isMobile}">
      <!-- 消息列表区 -->
      <div class="msg-list" ref="msgBox" @scroll="handleScroll">
        <div v-if="hasMore" class="load-more-wrap">
          <button @click="$emit('load-more')" :disabled="loadingMore" class="btn-load-epic">
            <span class="line left"></span>
            <span class="txt">{{ loadingMore ? '溯源中...' : '✦ 探寻更早的誓言 ✦' }}</span>
            <span class="line right"></span>
          </button>
        </div>
        
        <div v-if="messages.length === 0" class="empty-state">
          <div class="empty-icon">📜</div>
          <p>荒芜的岛屿正等待第一声誓言...</p>
        </div>

        <transition-group name="t-msg" tag="div" class="msg-group">
          <div v-for="msg in messages" :key="msg.id" 
               class="msg-item" 
               :class="{'msg-me': msg.user_id === user.id, 'msg-ai': !msg.user_id, 'msg-recalled': msg.is_recalled}">
            
            <div class="msg-av-wrap" v-if="msg.user_id !== user.id">
              <img :src="msg.user?.avatar || (msg.user_id ? defAv : dragonAv)" class="msg-av">
              <div class="av-glow" v-if="!msg.user_id"></div>
            </div>

            <div class="msg-content-wrap">
              <div class="msg-meta" v-if="msg.user_id !== user.id">
                <span class="m-name">{{ msg.user?.username || '龙主' }}</span>
                <span class="m-time">{{ fmtDate(msg.created_at) }}</span>
              </div>
              
              <div class="msg-bubble shadow-premium" :class="{'bubble-ai': !msg.user_id, 'bubble-special': msg.ai_interest}">
                <div v-if="msg.is_recalled" class="recalled-text">
                   <span class="ic">🕯️</span> 誓言已在因果中消散
                </div>
                <div v-else class="msg-body markdown-body" v-html="parseMd(msg.content)"></div>
                
                <div class="bubble-actions" v-if="msg.user_id === user.id && !msg.is_recalled">
                  <button class="btn-bubble-recall" @click="$emit('del-msg', msg.id)">撤回</button>
                </div>
                <div class="bubble-actions-ai" v-if="!msg.user_id && user.role==='admin'">
                   <button class="btn-resonance" @click="$emit('force-reply', msg.id)">强制共鸣</button>
                </div>
              </div>
            </div>

            <div class="msg-av-wrap" v-if="msg.user_id === user.id">
              <img :src="user.avatar || defAv" class="msg-av">
            </div>
          </div>
        </transition-group>
        <div class="scroll-anchor" ref="anchor"></div>
      </div>

      <!-- 底部交互区 -->
      <footer class="input-area glass-card" :class="{'is-disabled': !isLoggedIn}">
        <div class="input-container">
          <!-- 比例与档位调节 (Magic Settings) -->
          <transition name="pop">
            <div class="magic-settings-epic glass-card" v-if="showMagicSettings">
              <div class="s-group">
                <div class="s-label-epic"><span>法阵比例</span><small>SIZE</small></div>
                <div class="s-options-epic">
                  <button v-for="s in ratios" :key="s" 
                          :class="{active: imageSize===s}" @click="imageSize=s">{{ s }}</button>
                </div>
              </div>
              <div class="s-group">
                <div class="s-label-epic"><span>灵力档位</span><small>RESOLUTION</small></div>
                <div class="s-options-epic">
                  <button v-for="r in resolutions" :key="r" 
                          :class="{active: imageRes===r}" @click="imageRes=r">{{ r.toUpperCase() }}</button>
                </div>
              </div>
            </div>
          </transition>

          <div class="input-main">
            <div class="magic-entry-wrap">
              <button class="btn-text-act btn-toggle-settings" @click="showMagicSettings = !showMagicSettings" :class="{active: showMagicSettings}">
                调节
              </button>
            </div>

            <textarea 
              v-model="inputContent" 
              placeholder="镌刻誓言，或描述幻化的景象..." 
              @keydown.enter.prevent="handleEnter"
              :disabled="!isLoggedIn || isChecking || postCooldown > 0"
              rows="1"
              ref="inputRef"
              @input="autoResize"
            ></textarea>
            
            <div class="btn-group">
              <button class="btn-text-act btn-send-msg" 
                      :disabled="!inputContent.trim() || isChecking || postCooldown > 0" 
                      @click="handleSend">誓约</button>
              
              <button class="btn-text-act btn-gen-magic" 
                      :disabled="!inputContent.trim() || isChecking || postCooldown > 0" 
                      @click="handleGenerate">
                {{ isChecking ? '感应中...' : '幻化' }}
              </button>
            </div>
          </div>

          <div class="input-bottom-bar" v-if="isLoggedIn">
            <div class="cooldown-info" v-if="postCooldown > 0">
              <span class="cd-timer">灵力恢复中: {{ postCooldown }}s</span>
            </div>
            <div class="magic-hint-epic" v-else>
               神念聚焦：<span>{{ imageSize }}</span> 比例 / <span>{{ imageRes.toUpperCase() }}</span> 档位
            </div>
          </div>
        </div>

        <div class="login-mask" v-if="!isLoggedIn">
          <button class="btn-login-trigger" @click="$emit('open-modal', 'login')">登入龙屿以开启法阵</button>
        </div>
      </footer>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, watch } from 'vue';
import { marked } from 'marked';

const props = defineProps({
  messages: Array,
  hasMore: Boolean,
  loadingMore: Boolean,
  isLoggedIn: Boolean,
  user: Object,
  dragonAv: String,
  defAv: String,
  isChecking: Boolean,
  postCooldown: Number,
  isMobile: Boolean
});

const emit = defineEmits(['send', 'load-more', 'del-msg', 'force-reply', 'generate-image', 'open-modal', 'set-box-ref']);

const inputContent = ref('');
const msgBox = ref(null);
const anchor = ref(null);
const showMagicSettings = ref(false);
const imageSize = ref('1:1');
const imageRes = ref('1k');
const inputRef = ref(null);

const ratios = ['1:1', '16:9', '9:16', '3:2', '2:3', '4:3', '3:4'];
const resolutions = ['1k', '2k', '4k'];

const fmtDate = (t) => {
  const d = new Date(t);
  return `${d.getHours()}:${String(d.getMinutes()).padStart(2, '0')}`;
};

const parseMd = (content) => {
  if (!content) return '';
  return marked.parse(content);
};

const handleEnter = (e) => {
  if (e.shiftKey) return;
  handleSend();
};

const handleSend = () => {
  if (!inputContent.value.trim() || props.isChecking || props.postCooldown > 0) return;
  emit('send', inputContent.value);
  inputContent.value = '';
  resetTextarea();
};

const handleGenerate = () => {
  if (!inputContent.value.trim() || props.isChecking || props.postCooldown > 0) return;
  emit('generate-image', { 
    prompt: inputContent.value, 
    size: imageSize.value,
    resolution: imageRes.value
  });
  inputContent.value = '';
  showMagicSettings.value = false;
  resetTextarea();
};

const resetTextarea = () => {
  nextTick(() => { if(inputRef.value) { inputRef.value.style.height = 'auto'; } });
};

const autoResize = (e) => {
  e.target.style.height = 'auto';
  e.target.style.height = (e.target.scrollHeight) + 'px';
};

const scrollToBottom = (behavior = 'smooth') => {
  if (msgBox.value) {
    msgBox.value.scrollTo({ top: msgBox.value.scrollHeight, behavior });
  }
};

watch(() => props.messages.length, (newLen, oldLen) => {
  if (newLen > oldLen) {
    const isNearBottom = msgBox.value.scrollHeight - msgBox.value.scrollTop - msgBox.value.clientHeight < 250;
    if (isNearBottom || oldLen === 0) {
      nextTick(() => scrollToBottom('smooth'));
    }
  }
});

onMounted(() => {
  emit('set-box-ref', msgBox.value);
  nextTick(() => scrollToBottom('auto'));
});
</script>

<style scoped>
.view-chat { flex: 1; display: flex; flex-direction: column; overflow: hidden; background: transparent; }
.chat-area { flex: 1; display: flex; flex-direction: column; overflow: hidden; max-width: 1200px; width: 100%; margin: 0 auto; position: relative; }

.msg-list { flex: 1; overflow-y: auto; padding: 20px 40px 120px; scroll-behavior: smooth; contain: content; }

/* 探寻更早的誓言 美化 */
.load-more-wrap { display: flex; justify-content: center; margin-bottom: 40px; }
.btn-load-epic { 
  background: none; border: none; cursor: pointer; display: flex; align-items: center; gap: 20px; 
  color: #444; transition: .4s; width: 100%; max-width: 600px;
}
.btn-load-epic .line { height: 1px; flex: 1; background: linear-gradient(to right, transparent, rgba(255,255,255,0.05), transparent); transition: .4s; }
.btn-load-epic .txt { font-size: 0.75rem; letter-spacing: 4px; font-weight: bold; text-transform: uppercase; }
.btn-load-epic:hover { color: #888; }
.btn-load-epic:hover .line { background: linear-gradient(to right, transparent, rgba(192,57,43,0.3), transparent); flex: 1.5; }

.msg-group { display: flex; flex-direction: column; gap: 24px; }
.msg-item { display: flex; gap: 16px; max-width: 85%; align-self: flex-start; animation: msg-pop 0.4s cubic-bezier(0.16, 1, 0.3, 1) both; }
.msg-me { align-self: flex-end; flex-direction: row; }
.msg-av-wrap { flex-shrink: 0; width: 44px; height: 44px; position: relative; }
.msg-av { width: 100%; height: 100%; border-radius: 12px; object-fit: cover; border: 1px solid rgba(255,255,255,0.05); }

.msg-bubble { 
  padding: 16px 20px; border-radius: 20px; background: rgba(255,255,255,0.03); 
  border: 1px solid rgba(255,255,255,0.05); color: var(--text-main); line-height: 1.6; 
  backdrop-filter: blur(10px); position: relative;
}
.msg-me .msg-bubble { background: rgba(192,57,43,0.08); border-color: rgba(192,57,43,0.2); border-top-right-radius: 4px; }

/* 强制共鸣 美化 */
.bubble-actions-ai { margin-top: 10px; display: flex; justify-content: flex-start; }
.btn-resonance {
  background: rgba(192,57,43,0.05); border: 1px solid rgba(192,57,43,0.2); 
  color: #c0392b; font-size: 0.65rem; padding: 4px 12px; border-radius: 20px;
  cursor: pointer; transition: 0.3s; letter-spacing: 1px;
}
.btn-resonance:hover { 
  background: #c0392b; color: #fff; border-color: #c0392b; 
  box-shadow: 0 0 15px rgba(192,57,43,0.4);
}

:deep(.markdown-body img) {
  max-width: 400px; width: auto; height: auto; border-radius: 12px; margin: 10px 0;
  border: 1px solid rgba(255,255,255,0.1); box-shadow: 0 10px 30px rgba(0,0,0,0.3); display: block;
}

.btn-bubble-recall { 
  background: rgba(255,255,255,0.05); border: none; color: #666; 
  font-size: 0.65rem; padding: 4px 10px; border-radius: 6px; 
  cursor: pointer; transition: .3s; backdrop-filter: blur(5px);
}
.btn-bubble-recall:hover { color: #c0392b; background: rgba(192,57,43,0.1); }

/* 法阵调节面板 美化 */
.magic-settings-epic {
  position: absolute; bottom: calc(100% + 15px); left: 0; right: 0;
  padding: 30px; border-radius: 24px; background: rgba(10,10,10,0.98);
  box-shadow: 0 30px 60px rgba(0,0,0,0.6); z-index: 110; border: 1px solid rgba(255,255,255,0.05);
}
.s-label-epic { 
  display: flex; align-items: baseline; gap: 10px; margin-bottom: 16px; 
  padding-left: 10px; border-left: 2px solid #c0392b;
}
.s-label-epic span { font-size: 0.85rem; color: #eee; font-weight: bold; letter-spacing: 2px; }
.s-label-epic small { font-size: 0.6rem; color: #444; letter-spacing: 1px; font-weight: normal; }

.s-options-epic { display: flex; flex-wrap: wrap; gap: 10px; }
.s-options-epic button { 
  background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.05); 
  color: #666; padding: 8px 20px; border-radius: 30px; font-size: 0.8rem; 
  cursor: pointer; transition: 0.4s; font-weight: 500;
}
.s-options-epic button:hover { border-color: rgba(255,255,255,0.2); color: #aaa; }
.s-options-epic button.active { 
  background: #c0392b; border-color: #c0392b; color: #fff; 
  box-shadow: 0 5px 15px rgba(192,57,43,0.3);
}

.input-area { position: absolute; bottom: 20px; left: 40px; right: 40px; padding: 16px; border-radius: 28px; z-index: 100; }
.input-main { display: flex; gap: 12px; align-items: flex-end; }
textarea { 
  flex: 1; background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.1); 
  border-radius: 16px; color: #fff; padding: 12px 16px; font-size: 1rem; 
  resize: none; min-height: 48px; max-height: 150px; outline: none; transition: 0.3s;
}

.btn-group { flex: 1; display: flex; justify-content: flex-end; gap: 10px; align-items: center; }
.btn-text-act {
  flex: 1; min-width: 80px;
  background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08);
  color: #888; font-size: 0.85rem; font-weight: bold; height: 48px;
  border-radius: 14px; cursor: pointer; transition: 0.4s;
  display: flex; align-items: center; justify-content: center;
  white-space: nowrap; letter-spacing: 2px;
}
.btn-text-act:hover:not(:disabled) { background: rgba(255,255,255,0.08); color: #fff; transform: translateY(-2px); }
.btn-gen-magic { background: rgba(192,57,43,0.1); border-color: rgba(192,57,43,0.2); color: #c0392b; }
.btn-gen-magic:hover:not(:disabled) { background: #c0392b; color: #fff; border-color: #c0392b; box-shadow: 0 0 20px rgba(192,57,43,0.4); }

.magic-hint-epic { font-size: 0.65rem; color: #444; letter-spacing: 1px; font-weight: 500; }
.magic-hint-epic span { color: #888; font-weight: bold; }

@media (max-width: 768px) {
  .msg-list { padding: 20px 15px 140px; }
  .input-area { left: 10px; right: 10px; bottom: 10px; border-radius: 24px; padding: 12px; }
  .btn-text-act { padding: 0 12px; height: 42px; font-size: 0.75rem; }
  .magic-settings-epic { padding: 20px; }
}

@keyframes msg-pop { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
</style>
