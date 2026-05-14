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
          <div v-if="showManifesto" class="manifesto-overlay" @click.self="$emit('close-manifesto')">
            <div class="manifesto-card glass-card pop">
              <div class="mf-head">
                <div class="mf-title-wrap">
                  <span class="mf-icon">✦</span>
                  <h3>铸龙宣言 · 技术总览</h3>
                </div>
                <button class="btn-close" @click="$emit('close-manifesto')">×</button>
              </div>
              <div class="mf-body-wrap">
                <div class="mf-body markdown-body" v-html="parsedManifesto"></div>
              </div>
            </div>
          </div>
        </transition>

        <!-- 史诗内容列表 -->
        <div class="archives-container">
          <div v-if="archives.length===0" class="empty-tip">
            {{ activeTab === 0 ? '史官尚未落笔，岛屿尚在沉睡...' : '铸龙之锤尚未敲响，架构蓝图待绘...' }}
          </div>
          
          <transition-group name="t-arc">
            <div v-for="arc in archives" :key="arc.id" class="archive-card" :class="activeTab===1?'card-tech':'card-daily'">
              <div class="arc-header">
                <div class="arc-title-group">
                  <span v-if="activeTab===1" class="tech-tag">TECH</span>
                  <span class="arc-title">{{ arc.title }}</span>
                </div>
                <span class="arc-date">{{ arc.date }}</span>
              </div>
              <div class="arc-content markdown-body" v-html="parseMd(arc.content)"></div>
            </div>
          </transition-group>
        </div>

      </div>
    </section>
  </div>
</template>

<script setup>
import { reactive, ref, computed } from 'vue';
import { marked } from 'marked';

const props = defineProps({
  archives: Array,
  isAdmin: Boolean,
  activeTab: Number,
  manifesto: String,
  showManifesto: Boolean
});

const emit = defineEmits(['switch-tab', 'post-archive', 'show-manifesto', 'close-manifesto', 'analyze-tech']);

const isAnalyzing = ref(false);
const postForm = reactive({
  title: '',
  content: '',
  type: 1
});

// 解析 Markdown
const parseMd = (content) => {
  if (!content) return '';
  return marked.parse(content);
};

// 专门处理架构总览的解析
const parsedManifesto = computed(() => {
  return props.manifesto ? marked.parse(props.manifesto) : '';
});

const handleAnalyze = async () => {
  if (isAnalyzing.value) return;
  isAnalyzing.value = true;
  emit('analyze-tech', { title: postForm.title, content: postForm.content }, (suggestion) => {
    if(suggestion) {
      postForm.title = suggestion.version || suggestion.title || '';
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
.archive-card { 
  background: rgba(15,15,15,0.8); 
  border: 1px solid rgba(255,255,255,0.06); 
  border-radius: 24px; 
  padding: 40px; 
  margin-bottom: 40px; 
  transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); 
  transform: translateZ(0); 
  box-shadow: 0 10px 30px rgba(0,0,0,0.5); 
  will-change: transform, opacity;
}
.archive-card:hover { 
  border-color: rgba(192,57,43,0.6); 
  transform: translateY(-8px) scale(1.01); 
  box-shadow: 0 20px 40px rgba(0,0,0,0.8), 0 0 20px rgba(192,57,43,0.1);
}

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
.t-arc-enter-active { transition: all 0.8s cubic-bezier(0.16, 1, 0.3, 1); }
.t-arc-enter-from { opacity: 0; transform: translateY(40px) rotateX(-5deg); }
.t-arc-move { transition: transform 0.6s cubic-bezier(0.16, 1, 0.3, 1); }

.pop { animation: pop-in 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
@keyframes pop-in { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }

/* Manifesto Modal */
.manifesto-overlay { 
  position: fixed; inset: 0; background: rgba(0,0,0,0.88); 
  backdrop-filter: blur(40px); z-index: 1000; 
  display: flex; align-items: center; justify-content: center; padding: 40px; 
}
.manifesto-card { 
  width: 100%; max-width: 1000px; height: 100%; max-height: 85vh; 
  background: rgba(10,10,10,0.9); border: 1px solid rgba(255,255,255,0.08); 
  border-radius: 32px; display: flex; flex-direction: column; overflow: hidden; 
  box-shadow: 0 60px 120px rgba(0,0,0,1);
}
.mf-head { padding: 32px 48px; border-bottom: 1px solid rgba(255,255,255,0.05); display: flex; justify-content: space-between; align-items: center; }
.mf-title-wrap { display: flex; align-items: center; gap: 16px; }
.mf-icon { color: #c0392b; font-size: 1.2rem; filter: drop-shadow(0 0 8px #c0392b); }
.mf-head h3 { font-family: 'Noto Serif SC', serif; color: #fff; font-size: 1.4rem; letter-spacing: 4px; margin: 0; }
.btn-close { background: none; border: none; color: #444; font-size: 2.2rem; cursor: pointer; transition: .4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.btn-close:hover { color: #c0392b; transform: rotate(90deg) scale(1.2); }
.mf-body-wrap { flex: 1; overflow-y: auto; padding: 48px; scroll-behavior: smooth; }
.mf-body { background: transparent; }
.mf-body-wrap::-webkit-scrollbar { width: 4px; }
.mf-body-wrap::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.05); border-radius: 10px; }

/* Markdown Typography */
.markdown-body :deep(h1), .markdown-body :deep(h2), .markdown-body :deep(h3) { 
  font-family: 'Noto Serif SC', serif; 
  color: #fff; 
  margin: 24px 0 16px; 
  letter-spacing: 1px;
}
.markdown-body :deep(h1) { font-size: 1.8rem; border-bottom: 1px solid rgba(192,57,43,0.3); padding-bottom: 10px; }
.markdown-body :deep(h2) { font-size: 1.4rem; border-left: 4px solid #c0392b; padding-left: 15px; }
.markdown-body :deep(h3) { font-size: 1.1rem; color: #c0392b; }

.markdown-body :deep(p) { margin-bottom: 16px; line-height: 1.8; color: #bbb; }
.markdown-body :deep(strong) { color: #fff; font-weight: 700; }
.markdown-body :deep(ul), .markdown-body :deep(ol) { padding-left: 24px; margin-bottom: 16px; color: #aaa; }
.markdown-body :deep(li) { margin-bottom: 8px; }

.markdown-body :deep(code) { 
  background: rgba(192,57,43,0.1); 
  color: #ff6b6b; 
  padding: 2px 6px; 
  border-radius: 4px; 
  font-family: 'Consolas', monospace;
  font-size: 0.9em;
}

.markdown-body :deep(blockquote) {
  margin: 20px 0;
  padding: 10px 20px;
  background: rgba(255,255,255,0.02);
  border-left: 3px solid #444;
  font-style: italic;
  color: #888;
}

.markdown-body :deep(img) {
  max-width: 100%;
  height: auto;
  border-radius: 12px;
  margin: 10px 0;
}


.t-modal-enter-active, .t-modal-leave-active { transition: all 0.6s cubic-bezier(0.16, 1, 0.3, 1); }
.t-modal-enter-from, .t-modal-leave-to { opacity: 0; transform: scale(0.96) translateY(30px); }
/* --- 移动端适配 --- */
@media (max-width: 768px) {
  .archives-container { padding: 0 20px; }
  .admin-post-box { margin: 0 20px 40px; padding: 20px; }
  .manifesto-overlay { padding: 15px; }
  .manifesto-card { max-height: 92vh; border-radius: 20px; }
  .mf-head { padding: 20px; }
  .mf-head h3 { font-size: 1.1rem; letter-spacing: 2px; }
  .mf-body-wrap { padding: 24px; }
  .mf-body { font-size: 0.95rem; }
  .btn-close { font-size: 1.8rem; }
}
</style>
