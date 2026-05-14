<template>
  <div class="view-chat layout">
    <!-- 预览图层 -->
    <transition name="v-fade">
      <div v-if="previewImg" class="img-preview-overlay" @click="previewImg = null">
        <img :src="previewImg" class="preview-target" @click.stop>
        <div class="preview-hint">点击任意处返回 | 长按或右键可存图</div>
      </div>
    </transition>

    <section class="chat-area" :class="{'is-mobile': isMobile}">
      <!-- 消息列表区 -->
      <div class="msg-list" ref="msgBox" @scroll="handleScroll">
        <div v-if="hasMore" class="load-more-wrap">
          <button @click="$emit('load-more')" :disabled="loadingMore" class="btn-load-epic">
            <span class="line left"></span>
            <span class="txt">{{ loadingMore ? '溯源中...' : '探寻更早的誓言' }}</span>
            <span class="line right"></span>
          </button>
        </div>
        
        <div v-if="messages.length === 0" class="empty-state">
          <div class="empty-icon">◈</div>
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
              <div class="msg-meta" v-if="msg.user_id">
                <span class="m-name">{{ msg.user?.username || '游侠' }}</span>
                <span class="m-interest" :class="msg.ai_interest ? 'i-yes' : 'i-no'">
                  {{ msg.ai_interest ? '◈ 龙主颇感兴趣' : '◇ 龙主兴致缺缺' }}
                </span>
                <span class="m-time">{{ fmtDate(msg.created_at) }}</span>
              </div>
              <div class="msg-meta" v-else>
                <span class="m-name">龙主</span>
                <span class="m-time">{{ fmtDate(msg.created_at) }}</span>
              </div>
              
              <div class="msg-bubble shadow-premium" :class="{'bubble-ai': !msg.user_id, 'bubble-special': msg.ai_interest}">
                <div v-if="msg.is_recalled" class="recalled-text">
                   <span class="ic">◇</span> 誓言已在因果中消散
                </div>
                <div v-else class="msg-body markdown-body">
                  <div v-if="isMagicLoading(msg.content)" class="magic-loading-bubble">
                    <div class="magic-orb-mini"></div>
                    <span class="magic-loading-txt">{{ getLoadingText(msg.content) }}</span>
                  </div>
                  <div v-else v-html="parseMd(msg.content)" @click="handleBodyClick"></div>
                </div>
                
                <div class="bubble-actions" v-if="msg.user_id === user.id && !msg.is_recalled">
                  <button class="btn-bubble-recall" @click="$emit('del-msg', msg.id)">撤回</button>
                </div>
                <div class="bubble-actions-admin" v-if="msg.user_id && user.role==='admin' && !msg.ai_interest">
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
          <!-- 审核动画层 -->
          <transition name="v-fade">
            <div v-if="isVerifying" class="verifying-overlay">
              <div class="v-status-card" :class="verifyStatus">
                <div v-if="verifyStatus === 'checking'" class="v-orb"></div>
                <div v-if="verifyStatus === 'pass'" class="v-result-icon">✔</div>
                <div v-if="verifyStatus === 'fail'" class="v-result-icon">✘</div>
                <div class="v-text">{{ verifyText }}</div>
              </div>
            </div>
          </transition>
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
                  <div v-for="r in resolutions" :key="r" class="res-opt-wrap">
                    <button 
                            :class="{
                              active: imageRes===r, 
                              locked: (r==='2k' && !hasDragon) || (r==='4k' && !allTasksDone)
                            }" 
                            @click="handleSelectRes(r)">
                      {{ r.toUpperCase() }}
                      <span v-if="(r==='2k' && !hasDragon) || (r==='4k' && !allTasksDone)" class="lock-ic">🔒</span>
                    </button>
                    <div class="res-hint" v-if="(r==='2k' && !hasDragon)">需要契约龙嗣</div>
                    <div class="res-hint" v-if="(r==='4k' && !allTasksDone)">需完成今日修行</div>
                  </div>
                </div>
              </div>
              <div class="magic-confirm-row">
                <button class="btn-confirm-magic" @click="handleGenerate" :disabled="!inputContent.trim() || isChecking || postCooldown > 0">
                  开启幻化 <span class="m-count">{{ magicUsage }}/5</span>
                </button>
              </div>
            </div>
          </transition>

          <div class="input-main">


            <textarea 
              v-model="inputContent" 
              placeholder="镌刻誓言，或描述幻化的景象..." 
              @keydown.enter.prevent="handleEnter"
              :disabled="!isLoggedIn || isChecking || postCooldown > 0 || isVerifying"
              rows="1"
              ref="inputRef"
              @input="autoResize"
            ></textarea>
            
            <div class="btn-group">
              <button class="btn-text-act btn-send-msg" 
                      :disabled="!inputContent.trim() || isChecking || postCooldown > 0 || isVerifying" 
                      @click="handleSend">誓约</button>
              
              <button class="btn-text-act btn-gen-magic" 
                      :class="{ active: showMagicSettings }"
                      :disabled="isChecking || postCooldown > 0 || isVerifying" 
                      @click="showMagicSettings = !showMagicSettings">
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
  isMobile: Boolean,
  hasDragon: Boolean,
  allTasksDone: Boolean,
  magicUsage: Number
});

const emit = defineEmits(['send', 'load-more', 'del-msg', 'force-reply', 'generate-image', 'open-modal', 'set-box-ref']);

const inputContent = ref('');
const msgBox = ref(null);
const anchor = ref(null);
const showMagicSettings = ref(false);
const imageSize = ref('1:1');
const imageRes = ref('1k');
const inputRef = ref(null);

const previewImg = ref(null);
const isVerifying = ref(false);
const verifyStatus = ref('checking'); // checking, pass, fail
const verifyText = ref('正在通过龙语审核...');

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

// 监听内容区域点击，处理图片预览
const handleBodyClick = (e) => {
  if (e.target.tagName === 'IMG') {
    previewImg.value = e.target.src;
  }
};

const isMagicLoading = (content) => content && content.includes('[MAGIC_LOADING:');
const getLoadingText = (content) => {
  if (!content) return '正在凝聚灵力...';
  const match = content.match(/龙主正在凝聚灵力.../);
  return match ? match[0] : '正在凝聚灵力...';
};

const handleSelectRes = (r) => {
  if (r === '2k' && !props.hasDragon) return;
  if (r === '4k' && !props.allTasksDone) return;
  imageRes.value = r;
};

const handleEnter = (e) => {
  if (e.shiftKey) return;
  handleSend();
};

const handleSend = async () => {
  if (!inputContent.value.trim() || isVerifying.value) return;
  const content = inputContent.value;
  
  isVerifying.value = true;
  verifyStatus.value = 'checking';
  verifyText.value = '正在通过龙语审核...';

  try {
    emit('send', content, (success, err) => {
      if (success) {
        verifyStatus.value = 'pass';
        verifyText.value = '审核通过';
        setTimeout(() => {
          isVerifying.value = false;
          inputContent.value = '';
          resetTextarea();
        }, 1000);
      } else {
        verifyStatus.value = 'fail';
        verifyText.value = err || '含有违规内容';
        setTimeout(() => {
          isVerifying.value = false;
        }, 2000);
      }
    });
  } catch (e) {
    isVerifying.value = false;
  }
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

.m-interest { font-size: 0.65rem; padding: 2px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; }
.m-interest.i-yes { color: #c0392b; background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.2); }
.m-interest.i-no { color: #444; background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.05); }

.m-time { color: #444; font-size: 0.7rem; }
.msg-bubble { 
  max-width: 85%; padding: 16px 20px; border-radius: 24px; position: relative; 
  line-height: 1.6; word-break: break-word; font-size: 0.95rem; background: rgba(255,255,255,0.03); 
  border: 1px solid rgba(255,255,255,0.05); color: var(--text-main); backdrop-filter: blur(10px);
}
.bubble-actions-admin { position: absolute; left: calc(100% + 10px); top: 50%; transform: translateY(-50%); white-space: nowrap; }
.btn-resonance { 
  background: #c0392b; color: #fff; border: none; font-size: 0.65rem; padding: 6px 12px; 
  border-radius: 8px; cursor: pointer; transition: .3s; font-weight: bold; box-shadow: 0 4px 10px rgba(192,57,43,0.3);
}
.btn-resonance:hover { background: #e74c3c; transform: translateY(-2px); }

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

.s-options-epic { display: flex; gap: 8px; flex-wrap: wrap; }
.res-opt-wrap { position: relative; }
.res-hint { position: absolute; bottom: 100%; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.9); color: #ffd700; font-size: 0.6rem; padding: 4px 8px; border-radius: 4px; white-space: nowrap; visibility: hidden; opacity: 0; transition: 0.3s; pointer-events: none; margin-bottom: 8px; border: 1px solid #ffd700; }
.res-opt-wrap:hover .res-hint { visibility: visible; opacity: 1; }
.s-options-epic button { background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.1); color: #888; padding: 6px 16px; border-radius: 8px; cursor: pointer; transition: 0.3s; font-size: 0.75rem; position: relative; }
.s-options-epic button.locked { opacity: 0.5; cursor: not-allowed; border-style: dashed; }
.lock-ic { font-size: 0.6rem; margin-left: 4px; opacity: 0.8; }
.s-options-epic button.active { background: #c0392b; color: #fff; border-color: #c0392b; box-shadow: 0 0 15px rgba(192,57,43,0.4); }

:deep(.markdown-body img) {
  max-width: min(100%, 400px); height: auto; border-radius: 12px; margin: 10px 0;
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

.magic-confirm-row { margin-top: 30px; border-top: 1px solid rgba(255,255,255,0.05); padding-top: 25px; display: flex; justify-content: center; }
.btn-confirm-magic {
  width: 100%; max-width: 400px; height: 50px;
  background: linear-gradient(135deg, #c0392b, #8e44ad);
  color: #fff; border: none; border-radius: 16px;
  font-size: 1rem; font-weight: 900; letter-spacing: 4px;
  cursor: pointer; transition: 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  box-shadow: 0 10px 20px rgba(192,57,43,0.3);
  display: flex; align-items: center; justify-content: center; gap: 12px;
}
.btn-confirm-magic:hover:not(:disabled) { transform: scale(1.02) translateY(-2px); box-shadow: 0 15px 30px rgba(192,57,43,0.5); filter: brightness(1.1); }
.btn-confirm-magic:disabled { opacity: 0.3; cursor: not-allowed; filter: grayscale(1); }
.m-count { font-size: 0.75rem; opacity: 0.6; font-weight: normal; background: rgba(0,0,0,0.2); padding: 2px 8px; border-radius: 6px; }

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
  .msg-list { padding: 20px 10px 180px; }
  .input-area { left: 0; right: 0; bottom: 0; border-radius: 0; padding: 12px 15px calc(2px + env(safe-area-inset-bottom)); background: rgba(8,8,8,0.98); border-top: 1px solid rgba(192,57,43,0.3); box-shadow: 0 -10px 30px rgba(0,0,0,0.8); }
  .input-main { display: flex; flex-direction: column; gap: 8px; }
  textarea { width: 100%; font-size: 0.95rem; min-height: 40px; }
  .btn-group { width: 100%; display: flex; gap: 8px; }
  .btn-text-act { flex: 1; height: 38px; font-size: 0.75rem; border-radius: 10px; }
  .magic-settings-epic { padding: 15px; border-radius: 16px 16px 0 0; bottom: 100%; left: 0; right: 0; }
  .s-label-epic { margin-bottom: 12px; }
  .s-options-epic button { padding: 6px 12px; font-size: 0.7rem; }
}

@keyframes msg-pop { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
/* 魔法加载气泡 */
.magic-loading-bubble { display: flex; align-items: center; gap: 12px; padding: 4px 0; }
.magic-orb-mini {
  width: 16px; height: 16px; 
  background: radial-gradient(circle, #ff3c00 0%, #c0392b 70%);
  border-radius: 50%; box-shadow: 0 0 10px #c0392b;
  animation: magic-orb-pulse 1s infinite alternate ease-in-out;
}
.magic-loading-txt { color: #c0392b; font-weight: bold; font-size: 0.9rem; letter-spacing: 2px; }

@keyframes magic-orb-pulse {
  from { transform: scale(0.8); opacity: 0.6; box-shadow: 0 0 5px #c0392b; }
  to { transform: scale(1.2); opacity: 1; box-shadow: 0 0 15px #c0392b; }
}
/* 审核动画 */
.verifying-overlay {
  position: absolute; inset: 0; background: rgba(0,0,0,0.8);
  backdrop-filter: blur(10px); z-index: 100; border-radius: 20px;
  display: flex; align-items: center; justify-content: center;
}
.v-status-card { text-align: center; }
.v-orb {
  width: 40px; height: 40px; border: 3px solid rgba(192,57,43,0.3);
  border-top-color: #c0392b; border-radius: 50%; margin: 0 auto 15px;
  animation: v-spin 1s linear infinite;
}
@keyframes v-spin { to { transform: rotate(360deg); } }

.v-result-icon { font-size: 2.5rem; margin-bottom: 10px; line-height: 1; }
.pass .v-result-icon { color: #2ecc71; animation: v-pop .3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.fail .v-result-icon { color: #e74c3c; animation: v-pop .3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
@keyframes v-pop { from { transform: scale(0.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }

.v-text { font-size: 0.9rem; letter-spacing: 2px; color: #aaa; }
.pass .v-text { color: #2ecc71; }
.fail .v-text { color: #e74c3c; }

.v-fade-enter-active, .v-fade-leave-active { transition: opacity 0.3s; }
.v-fade-enter-from, .v-fade-leave-to { opacity: 0; }

.btn-bubble-recall {
  background: rgba(255,255,255,0.05); color: #666; border: 1px solid rgba(255,255,255,0.1);
  font-size: 0.65rem; padding: 4px 10px; border-radius: 6px; cursor: pointer; transition: .3s;
}

.magic-count { font-size: 0.65rem; opacity: 0.6; margin-left: 4px; font-weight: normal; background: rgba(0,0,0,0.3); padding: 1px 4px; border-radius: 4px; }
.btn-gen-magic:hover .magic-count { opacity: 1; color: #fff; }
.btn-bubble-recall:hover { color: #c0392b; background: rgba(192,57,43,0.1); border-color: #c0392b; }
.bubble-actions { position: absolute; right: calc(100% + 10px); top: 50%; transform: translateY(-50%); white-space: nowrap; }

/* 图片预览层 */
.img-preview-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.95); z-index: 9999;
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  backdrop-filter: blur(20px);
}
.preview-target { max-width: 95vw; max-height: 85vh; border-radius: 12px; box-shadow: 0 0 50px rgba(0,0,0,0.5); object-fit: contain; }
.preview-hint { margin-top: 20px; color: #444; font-size: 0.8rem; letter-spacing: 2px; }
</style>
