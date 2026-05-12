<template>
  <div class="view-archives layout">
    <section class="chat-area">
      <div class="msg-list">
        
        <!-- 史诗页签导航 -->
        <nav class="arc-tabs">
          <div class="arc-tab" :class="{active: activeTab===0}" @click="handleTabSwitch(0)">
            <span class="tab-en">Island Chronicles</span>
            <span class="tab-zh">岛屿行纪</span>
          </div>
          <div class="arc-tab" :class="{active: activeTab===1}" @click="handleTabSwitch(1)">
            <span class="tab-en">Dragon Codex</span>
            <span class="tab-zh">铸龙图谱</span>
          </div>
          <button class="btn-manifesto" @click="$emit('show-manifesto')">✦ 架构总览</button>
        </nav>

        <!-- 管理员发布表单 (仅在图谱页且是管理员时显示) -->
        <div v-if="activeTab===1 && isAdmin" class="admin-post-box pop">
          <div class="post-header">
            <span>✦ 铸刻新的技术进展</span>
            <button class="btn-ai-analyze" @click="handleAnalyze" :disabled="isAnalyzing">
              {{ isAnalyzing ? '正在扫描龙骨...' : '✨ 唤醒架构师建议' }}
            </button>
          </div>
          <input v-model="postForm.title" placeholder="版本号或进展标题 (如: v1.1.0 架构升级)" class="post-inp">
          <textarea v-model="postForm.content" placeholder="详细描述这次演进的内容..." class="post-txt"></textarea>
          <div class="post-actions">
            <button class="btn-p" @click="submitPost" :disabled="!postForm.title || !postForm.content">铸刻记录</button>
          </div>
        </div>

        <!-- 架构总览浮层 -->
        <transition name="t-modal">
          <div v-if="manifesto" class="manifesto-overlay" @click.self="$emit('close-manifesto')">
            <div class="manifesto-card">
              <div class="mf-head">
                <h3>铸龙宣言 · 技术总览</h3>
                <button class="btn-close" @click="$emit('close-manifesto')">×</button>
              </div>
              <pre class="mf-body">{{ manifesto }}</pre>
            </div>
          </div>
        </transition>

        <!-- 史诗内容列表 -->
        <div class="archives-container">
          <div v-if="archives.length===0" class="empty-tip">
            {{ activeTab === 0 ? '史官尚未落笔，岛屿尚在沉睡...' : '铸龙之锤尚未敲响，架构蓝图待绘...' }}
          </div>
          
          <transition-group name="t-arc">
            <div v-for="arc in archives" :key="arc.ID" class="archive-card" :class="activeTab===1?'card-tech':'card-daily'">
              <div class="arc-header">
                <div class="arc-title-group">
                  <span v-if="activeTab===1" class="tech-tag">TECH</span>
                  <span class="arc-title">{{ arc.title }}</span>
                </div>
                <span class="arc-date">{{ arc.date }}</span>
              </div>
              <div class="arc-content" v-html="arc.content"></div>
            </div>
          </transition-group>
        </div>

      </div>
    </section>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue';

const props = defineProps({
  archives: Array,
  isAdmin: Boolean,
  activeTab: Number,
  manifesto: String
});

const emit = defineEmits(['switch-tab', 'post-archive', 'show-manifesto', 'close-manifesto', 'analyze-tech']);

const isAnalyzing = ref(false);
const postForm = reactive({
  title: '',
  content: '',
  type: 1
});

const handleAnalyze = async () => {
  isAnalyzing.value = true;
  emit('analyze-tech', (suggestion) => {
    if(suggestion) {
      postForm.title = suggestion.title || '';
      postForm.content = suggestion.content || '';
    }
    isAnalyzing.value = false;
  });
};

const handleTabSwitch = (t) => {
  emit('switch-tab', t);
};

const submitPost = () => {
  emit('post-archive', { ...postForm });
  postForm.title = '';
  postForm.content = '';
};
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;background: transparent;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;background: linear-gradient(to right, rgba(0,0,0,0.85) 0%, rgba(0,0,0,0.6) 50%, rgba(0,0,0,0.85) 100%); transform: translateZ(0); will-change: transform;}
.msg-list{flex:1;overflow-y:auto;padding:60px 0;background: transparent;}

/* Tabs Styling */
.arc-tabs { display: flex; gap: 40px; margin-bottom: 60px; padding: 0 60px; border-bottom: 1px solid rgba(255,255,255,0.05); }
.arc-tab { padding-bottom: 15px; cursor: pointer; transition: .3s; position: relative; display: flex; flex-direction: column; }
.tab-en { font-size: 0.65rem; color: #444; letter-spacing: 2px; text-transform: uppercase; margin-bottom: 4px; }
.tab-zh { font-size: 1.1rem; color: #777; letter-spacing: 4px; font-family: 'Noto Serif SC', serif; }
.arc-tab:hover .tab-zh { color: #aaa; }
.arc-tab.active .tab-zh { color: #fff; }
.arc-tab.active .tab-en { color: #c0392b; }
.arc-tab.active::after { content: ''; position: absolute; bottom: -1px; left: 0; width: 100%; height: 2px; background: #c0392b; box-shadow: 0 0 10px #c0392b; }

.btn-manifesto { background: none; border: 1px solid #333; color: #666; font-size: 0.75rem; padding: 4px 12px; border-radius: 6px; height: fit-content; align-self: flex-end; margin-bottom: 12px; cursor: pointer; transition: .3s; }
.btn-manifesto:hover { border-color: #c0392b; color: #c0392b; }

.archives-container { padding: 0 60px; }

/* Admin Post Form */
.admin-post-box { background: rgba(255,255,255,0.02); border: 1px solid rgba(192,57,43,0.3); border-radius: 20px; padding: 30px; margin: 0 60px 60px; backdrop-filter: blur(10px); }
.post-header { display: flex; justify-content: space-between; align-items: center; font-size: 0.9rem; color: #c0392b; font-weight: bold; margin-bottom: 20px; letter-spacing: 2px; }
.btn-ai-analyze { background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #c0392b; font-size: 0.7rem; padding: 4px 12px; border-radius: 8px; cursor: pointer; transition: .3s; }
.btn-ai-analyze:hover:not(:disabled) { background: #c0392b; color: #fff; box-shadow: 0 0 15px rgba(192,57,43,0.4); }
.btn-ai-analyze:disabled { opacity: 0.5; cursor: wait; }
.post-inp { width: 100%; background: rgba(0,0,0,0.3); border: 1px solid rgba(255,255,255,0.1); border-radius: 10px; padding: 12px 16px; color: #fff; margin-bottom: 16px; outline: none; }
.post-txt { width: 100%; height: 120px; background: rgba(0,0,0,0.3); border: 1px solid rgba(255,255,255,0.1); border-radius: 10px; padding: 16px; color: #fff; margin-bottom: 16px; outline: none; resize: none; font-family: inherit; }
.post-actions { text-align: right; }
.btn-p { background: #c0392b; color: #fff; border: none; border-radius: 8px; padding: 10px 30px; font-weight: bold; cursor: pointer; transition: .2s; }
.btn-p:hover:not(:disabled) { background: #e74c3c; transform: translateY(-2px); }
.btn-p:disabled { opacity: 0.3; }

/* Archive Cards */
.archive-card { background: rgba(15,15,15,0.8); border: 1px solid rgba(255,255,255,0.06); border-radius: 20px; padding: 40px; margin-bottom: 40px; transition: .4s; transform: translateZ(0); box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
.archive-card:hover { border-color: rgba(192,57,43,0.4); transform: translateY(-4px); }

/* Daily Style */
.card-daily { border-left: 1px solid rgba(192,57,43,0.2); }

/* Tech Style */
.card-tech { border-left: 4px solid #2980b9; background: linear-gradient(135deg, rgba(15,15,15,0.95) 0%, rgba(10,30,50,0.5) 100%); }
.tech-tag { background: #2980b9; color: #fff; font-size: 0.6rem; padding: 2px 6px; border-radius: 4px; margin-right: 12px; vertical-align: middle; }
.card-tech .arc-title { font-family: 'Inter', sans-serif; font-weight: 700; color: #3498db; }

.arc-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; border-bottom: 1px solid rgba(255,255,255,0.05); padding-bottom: 20px; }
.arc-title { font-family: 'Noto Serif SC', serif; font-size: 1.6rem; color: #f0f0f0; letter-spacing: 2px; }
.arc-date { font-size: .85rem; color: #444; font-family: 'Inter'; }
.arc-content { font-size: 1.05rem; color: #bbb; line-height: 1.8; white-space: pre-wrap; letter-spacing: 0.5px; }

.empty-tip { text-align: center; color: #222; padding: 100px 0; letter-spacing: 4px; font-size: 1.1rem; }

/* Transitions */
.t-arc-enter-active { transition: all 0.6s ease; }
.t-arc-enter-from { opacity: 0; transform: translateX(30px); }

.pop { animation: pop-in 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
@keyframes pop-in { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }

/* Manifesto Modal */
.manifesto-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.85); backdrop-filter: blur(15px); z-index: 1000; display: flex; align-items: center; justify-content: center; padding: 40px; }
.manifesto-card { width: 100%; max-width: 900px; height: 100%; max-height: 80vh; background: #080808; border: 1px solid rgba(255,255,255,0.1); border-radius: 24px; display: flex; flex-direction: column; overflow: hidden; box-shadow: 0 30px 60px rgba(0,0,0,0.8); }
.mf-head { padding: 24px 32px; border-bottom: 1px solid rgba(255,255,255,0.05); display: flex; justify-content: space-between; align-items: center; }
.mf-head h3 { font-family: 'Noto Serif SC', serif; color: #fff; font-size: 1.2rem; letter-spacing: 2px; }
.btn-close { background: none; border: none; color: #444; font-size: 2rem; cursor: pointer; transition: .2s; }
.btn-close:hover { color: #c0392b; }
.mf-body { flex: 1; overflow-y: auto; padding: 32px; font-family: 'Consolas', 'Monaco', monospace; color: #aaa; font-size: 0.95rem; line-height: 1.8; white-space: pre-wrap; background: rgba(255,255,255,0.01); }
.mf-body::-webkit-scrollbar { width: 4px; }
.mf-body::-webkit-scrollbar-thumb { background: #1a1a1a; }

.t-modal-enter-active, .t-modal-leave-active { transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.t-modal-enter-from, .t-modal-leave-to { opacity: 0; transform: scale(0.9) translateY(20px); }
</style>
