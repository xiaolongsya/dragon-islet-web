<template>
  <div class="app" :class="{'is-mobile': isMobile}">
    <!-- 全局导航 -->
    <GlobalNav 
      v-model="currentView" 
      :isLoggedIn="isLoggedIn" 
      :user="user" 
      :defAv="defAv"
      :isMobile="isMobile"
      @open-modal="openModal" 
    />

    <main class="main-view">
      <transition name="t-view" mode="out-in">
        <HomeView 
          v-if="currentView==='home'"
          :isLoggedIn="isLoggedIn"
          :isThinking="isThinking"
          :thinkingText="thinkingText"
          :quote="quote"
          :displayedQuote="displayedQuote"
          :showExplain="showExplain"
          :showEnter="showEnter"
          :isShaking="isShaking"
          @enter="currentView='chat'"
          @show-about="currentView='about'"
          @get-fortune="fetchFortune"
        />

        <ChatView 
          v-else-if="currentView==='chat'"
          :messages="messages"
          :hasMore="hasMore"
          :loadingMore="loadingMore"
          :isLoggedIn="isLoggedIn"
          :user="user"
          :dragonAv="dragonAv"
          :defAv="defAv"
          :isChecking="isChecking"
          :postCooldown="postCooldown"
          :isMobile="isMobile"
          @send="send"
          @load-more="loadMore"
          @del-msg="delMsg"
          @force-reply="forceReply"
          @generate-image="generateImage"
          @open-modal="openModal"
          @set-box-ref="val => msgBox = val"
        />

        <ArchiveView 
          v-else-if="currentView==='archives'"
          :archives="archives"
          :isAdmin="user.role==='admin'"
          :activeTab="activeArchiveTab"
          :manifesto="manifestoContent"
          @switch-tab="switchArchiveTab"
          @post-archive="postArchive"
          @show-manifesto="fetchManifesto"
          @close-manifesto="manifestoContent=''"
          @analyze-tech="analyzeTech"
        />

        <MyOathsView 
          v-else-if="currentView==='my-oaths'"
          :items="myOaths"
          :total="myOathsTotal"
          :page="myOathsPage"
          :limit="myOathsLimit"
          :feedbacks="myFeedbacks"
          :user="user"
          @change-page="p => { myOathsPage=p; fetchMyOaths(); }"
          @del-msg="delMyMsg"
          @del-fb="delFB"
          @open-profile-modal="openModal('profile')"
        />

        <FeedbackView 
          v-else-if="currentView==='feedback'"
          @submit="submitFB"
        />

        <AdminView 
          v-else-if="currentView==='admin'"
          :items="adminFeedbacks"
          :total="adminTotal"
          :page="adminPage"
          :limit="adminLimit"
          @reply="replyFB"
          @change-page="p => { adminPage=p; fetchAdminFeedbacks(); }"
          @manual-generate="manualGenerateEpic"
          :isGenerating="isGeneratingEpic"
        />

        <RaisingView v-else-if="currentView==='raising'" :isLoggedIn="isLoggedIn" />

        <AboutView v-else-if="currentView==='about'" />
      </transition>
    </main>

    <!-- 弹窗层 -->
    <div class="modal-layer">
      <!-- 灵语签文 (Fortune Result) -->
      <transition name="pop">
        <div class="fortune-overlay" v-if="fortuneResult" @click="fortuneResult = null">
          <div class="fortune-card glass-card pop" @click.stop>
            <div class="f-luck">{{ fortuneResult.luck || fortuneResult.Luck || '平' }}</div>
            <div class="f-title">{{ fortuneResult.verse || fortuneResult.Verse || '灵力感应' }}</div>
            <div class="f-div"></div>
            <div class="f-interpretation">
              {{ fortuneResult.interpretation || fortuneResult.Interpretation || '神龙在云端低语，请静候灵旨。' }}
            </div>
            
            <div class="f-grid">
              <div class="f-box suit">
                <div class="label">宜 · SUIT</div>
                <div class="items-list">
                  <div v-for="it in ((fortuneResult.suit || fortuneResult.Suit || '').split(',') || [])" :key="it" class="it-row">
                    <span v-if="it">✦ {{ it }}</span>
                  </div>
                </div>
              </div>
              <div class="f-box avoid">
                <div class="label">忌 · AVOID</div>
                <div class="items-list">
                  <div v-for="it in ((fortuneResult.avoid || fortuneResult.Avoid || '').split(',') || [])" :key="it" class="it-row">
                    <span v-if="it">✦ {{ it }}</span>
                  </div>
                </div>
              </div>
            </div>

            <button class="btn-close-f" @click="fortuneResult = null">领受灵旨</button>
          </div>
        </div>
      </transition>

      <transition name="pop">
        <AuthModal 
          v-if="modal==='login' || modal==='register'"
          :type="modal"
          :form="authForm"
          :smsCooldown="smsCooldown"
          @close="modal=''"
          @submit="modal==='login'?doLogin():doRegister()"
          @switch="modal = modal==='login'?'register':'login'"
          @send-sms="sendSms"
        />
      </transition>

      <transition name="pop">
        <ProfileModal 
          v-if="modal==='profile'"
          :user="user"
          :editForm="editForm"
          :defAv="defAv"
          @close="modal=''"
          @submit="updateProfile"
          @logout="logout"
          @upload-avatar="uploadAvatar"
        />
      </transition>
    </div>

    <!-- 审核遮罩 -->
    <div class="audit-mask" v-if="isChecking">
      <div class="audit-box pop">
        <div class="audit-visual" :class="moderationStatus">
          <div class="spin-ring"></div>
          <div class="audit-status-icon" v-if="moderationStatus!=='examining'">
            {{ moderationStatus.startsWith('pass') ? '✓' : '✕' }}
          </div>
        </div>
        <div class="al">{{ auditText }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, reactive, watch, computed, onBeforeUnmount } from 'vue';
import axios from 'axios';

import GlobalNav from './components/GlobalNav.vue';
import AuthModal from './components/AuthModal.vue';
import ProfileModal from './components/ProfileModal.vue';
import HomeView from './views/HomeView.vue';
import ChatView from './views/ChatView.vue';
import MyOathsView from './views/MyOathsView.vue';
import ArchiveView from './views/ArchiveView.vue';
import FeedbackView from './views/FeedbackView.vue';
import AdminView from './views/AdminView.vue';
import AboutView from './views/AboutView.vue';
import RaisingView from './views/RaisingView.vue';

axios.defaults.baseURL = import.meta.env.VITE_API_BASE_URL;
axios.interceptors.request.use(c => { const t = localStorage.getItem('token'); if(t) c.headers.Authorization=`Bearer ${t}`; return c; });

const currentView = ref('home');
const isThinking = ref(false);
const quote = ref({ quote: '山重水复疑无路，柳暗花明又一村。', explain: '欢迎来到龙屿遗境。', type: 'wisdom' });
const displayedQuote = ref('山重水复疑无路，柳暗花明又一村。');
const showExplain = ref(true);
const showEnter = ref(true);
const thinkingTexts = ['龙主正在云端思志...', '扭动尾巴，凝谈道的精华...', '龙鲱翻涌，智慧正在凝聚...', '刚刺一裂云隙，真言将出...'];
const thinkingText = ref(thinkingTexts[0]);
let thinkingTimer = null;

const messages = ref([]); const archives = ref([]); const isChecking = ref(false);
const isLoggedIn = ref(!!localStorage.getItem('token'));
const user = ref(JSON.parse(localStorage.getItem('user')||'{}')); 
const postCooldown = ref(0); const msgBox = ref(null); const moderationStatus = ref(''); 
const auditText = computed(() => {
  if (moderationStatus.value.startsWith('pass')) return '✦ 龙语审阅：通过 ✦';
  if (moderationStatus.value === 'fail') return '✦ 龙语审阅：驳回 ✦';
  return '龙主正在审阅你的誓言...';
});
const modal = ref(''); const hasMore = ref(false); const loadingMore = ref(false);
const oldestID = ref(0); const smsCooldown = ref(0);
const isGeneratingEpic = ref(false);
const authForm = reactive({username:'',password:'',phone:'',code:''});
const editForm = reactive({username:'',avatar:'',motto:''});
const myOaths = ref([]); const myOathsTotal = ref(0); const myOathsPage = ref(1); const myOathsLimit = 10;
const myFeedbacks = ref([]);
const adminFeedbacks = ref([]); const adminTotal = ref(0); const adminPage = ref(1); const adminLimit = 10;
const activeArchiveTab = ref(0);
const manifestoContent = ref('');
const isMobile = ref(window.innerWidth <= 768);
const fortuneResult = ref(null); // 存放求签结果

const defAv = `https://xiaolongya.cn/uploads/1778432333617872906.jpg`;
const dragonAv = `https://xiaolongya.cn/uploads/1778566530694964727.jpg`;

const handleResize = () => { isMobile.value = window.innerWidth <= 768; };
window.addEventListener('resize', handleResize);

const isShaking = ref(false);
const fetchFortune = async () => {
  if(!isLoggedIn.value) return alert('请先建立契约（登录）再求取灵语');
  if(isShaking.value) return;
  isShaking.value = true;
  try {
    await new Promise(r => setTimeout(r, 1500));
    const r = await axios.get('/user/fortune');
    if (r.data && r.data.data) {
      fortuneResult.value = r.data.data;
    } else if (r.data) {
      fortuneResult.value = r.data;
    }
  } catch(e) {
    alert(e.response?.data?.error || '今日灵力波动不稳，请稍后再试');
  } finally {
    isShaking.value = false;
  }
};

const fetchQuote = async () => {
  startThinkingRotation();
  try { const r = await axios.get('/quote'); quote.value = r.data; } 
  catch { quote.value = { quote: '山知道我，我知道你，就已足够。', explain: '翻译成大白话：不需要全世界都懂我，你懂就够了。', type: 'wisdom' }; }
  clearInterval(thinkingTimer); isThinking.value = false;
  setTimeout(() => { runTypewriter(quote.value.quote, () => { setTimeout(() => { showExplain.value = true; }, 600); setTimeout(() => { showEnter.value = true; }, 1200); }); }, 400);
};

const openModal = (m) => { if(m==='profile'){editForm.username=user.value.username;editForm.avatar=user.value.avatar||'';editForm.motto=user.value.motto||'';} authForm.username='';authForm.password='';authForm.phone='';authForm.code=''; modal.value=m; };

const doLogin = async () => { try { const r=await axios.post('/auth/login',authForm); localStorage.setItem('token',r.data.token); localStorage.setItem('user',JSON.stringify(r.data.user)); isLoggedIn.value=true; user.value=r.data.user; modal.value=''; } catch(e){ alert(e.response?.data?.error||'登录失败'); }};
const doRegister = async () => { try { await axios.post('/auth/register',authForm); alert('注册成功'); modal.value='login'; } catch(e){ alert(e.response?.data?.error||'失败'); }};
const sendSms = async () => { if(!authForm.phone) return alert('请输入手机号'); try { await axios.post('/auth/send-sms',{phone:authForm.phone}); smsCooldown.value=60; const ti=setInterval(()=>{ if(smsCooldown.value>0)smsCooldown.value--; else clearInterval(ti); },1000); } catch(e){ alert(e.response?.data?.error||'发送失败'); }};

const updateProfile = async () => { try { await axios.post('/user/profile',{nickname:editForm.username,avatar:editForm.avatar,motto:editForm.motto}); user.value.username=editForm.username; user.value.motto=editForm.motto; if(editForm.avatar) user.value.avatar=editForm.avatar; localStorage.setItem('user',JSON.stringify(user.value)); modal.value=''; } catch(e){ alert(e.response?.data?.error||'失败'); }};
const uploadAvatar = async (e) => { const f=e.target.files[0]; if(!f) return; if(f.size>5*1024*1024) return alert('图片不能超过5MB'); const fd=new FormData(); fd.append('file',f); try { const r=await axios.post('/upload',fd); if(r.data.code===0) editForm.avatar=r.data.data.url; else alert(r.data.msg); } catch(err){ alert(err.response?.data?.msg||'上传失败'); }};
const logout = () => { localStorage.clear(); window.location.reload(); };

const fetchUserInfo = async () => {
  if(!isLoggedIn.value) return;
  try {
    const r = await axios.get('/user/profile');
    user.value = r.data;
    localStorage.setItem('user', JSON.stringify(user.value));
  } catch(e) { console.error('同步用户信息失败', e); }
};

const send = async (content) => {
  moderationStatus.value='examining'; isChecking.value=true;
  try {
    const r=await axios.post('/chat/send',{content});
    moderationStatus.value=r.data.will_reply?'pass-interested':'pass-ignored';
    setTimeout(fetchUserInfo, 1000);
    setTimeout(()=>{ isChecking.value=false; moderationStatus.value=''; },1400);
    postCooldown.value=60; const ti=setInterval(()=>{ if(postCooldown.value>0)postCooldown.value--; else clearInterval(ti); },1000);
  } catch(e) {
    moderationStatus.value='fail';
    setTimeout(()=>{ isChecking.value=false; moderationStatus.value=''; alert(e.response?.data?.error||'被拦截'); },1500);
  }
};

const fetchMessages = async () => {
  const r = await axios.get('/chat/list');
  // 核心：时间晚的在下面，所以我们要反转 desc 获取的列表
  messages.value = r.data.data.reverse();
  hasMore.value = r.data.has_more;
  if(messages.value.length>0) oldestID.value = messages.value[0].id;
};

const loadMore = async () => {
  if(loadingMore.value||!hasMore.value) return;
  loadingMore.value = true;
  try {
    const r = await axios.get(`/chat/list?before_id=${oldestID.value}`);
    const older = r.data.data.reverse(); // 获取更早的消息，也要反转后放在顶部
    messages.value = [...older, ...messages.value];
    hasMore.value = r.data.has_more;
    if(older.length>0) oldestID.value = older[0].id;
  } finally { loadingMore.value=false; }
};

const fetchArchives = async (type = 0) => {
  try {
    const r = await axios.get(`/archives?type=${type}`);
    archives.value = r.data.data;
  } catch(e) { console.error('获取史诗失败', e); }
};

const switchArchiveTab = (type) => { activeArchiveTab.value = type; fetchArchives(type); };

const postArchive = async (data) => {
  try { await axios.post('/archives', data); alert('铸龙图谱已更新'); fetchArchives(1); } 
  catch(e) { alert(e.response?.data?.error || '发布失败'); }
};

const fetchManifesto = async () => {
  try { const r = await axios.get('/archives/manifesto'); manifestoContent.value = r.data.content; } 
  catch(e) { alert('获取总览失败'); }
};

const analyzeTech = async (callback) => {
  try { const r = await axios.get('/archives/analyze'); callback(r.data); } 
  catch(e) { alert(e.response?.data?.error || '分析失败'); callback(null); }
};

const generateImage = async ({ prompt, size, resolution }) => {
  try { 
    const r = await axios.post('/chat/generate-image', { prompt, size, resolution }); 
    alert(r.data.message); 
  } 
  catch(e) { alert(e.response?.data?.error || '幻化失败'); }
};

const delMsg = async (id) => {
  if(!confirm('确定要撤回这条誓言吗？')) return;
  try { await axios.delete(`/chat/${id}`); const m = messages.value.find(x=>x.id===id); if(m) m.is_recalled = true; } 
  catch(e) { alert(e.response?.data?.error || '撤回失败'); }
};

const initWS = () => { 
  const ws=new WebSocket(import.meta.env.VITE_WS_URL); 
  ws.onmessage=(e)=>{ 
    const m=JSON.parse(e.data); const idx = messages.value.findIndex(x=>x.id===m.id);
    if(idx !== -1) messages.value[idx] = m; else messages.value.push(m); // 核心：新消息放在底部
  }; 
  ws.onclose=()=>setTimeout(initWS,3000); 
};

const fetchMyOaths = async () => { try { const r = await axios.get(`/chat/my?page=${myOathsPage.value}&limit=${myOathsLimit}`); myOaths.value = r.data.data; myOathsTotal.value = r.data.total; } catch(e) { console.error(e); }};
const fetchMyFeedbacks = async () => { try { const r = await axios.get('/feedback/my'); myFeedbacks.value = r.data.data; } catch {} };
const delMyMsg = async (id) => { if(!confirm('确定要抹除这条誓言吗？')) return; try { await axios.delete(`/chat/${id}`); myOaths.value = myOaths.value.filter(m => m.id !== id); myOathsTotal.value--; } catch(e) { alert(e.response?.data?.error || '抹除失败'); }};
const delFB = async (id) => { if(!confirm('确定要抹除这份信笺吗？')) return; try { await axios.delete(`/feedback/${id}`); myFeedbacks.value = myFeedbacks.value.filter(x => x.id !== id); } catch(e) { alert(e.response?.data?.error || '抹除失败'); }};

const submitFB = async (content) => { try { await axios.post('/feedback/submit', { content }); alert('信笺已投递'); currentView.value = 'my-oaths'; fetchMyFeedbacks(); } catch (e) { alert(e.response?.data?.error || '投递失败'); }};

const fetchAdminFeedbacks = async () => { try { const r = await axios.get(`/admin/feedback?page=${adminPage.value}&limit=${adminLimit}`); adminFeedbacks.value = r.data.data.map(fb => ({...fb, replyInput: ''})); adminTotal.value = r.data.total; } catch(e) { alert(e.response?.data?.error || '获取失败'); }};
const replyFB = async (fb) => { if(!fb.replyInput.trim()) return; try { await axios.post('/admin/feedback/reply', { id: fb.id, content: fb.replyInput }); fb.is_replied = true; fb.reply_content = fb.replyInput; alert('回响已传达'); } catch(e) { alert(e.response?.data?.error || '回复失败'); }};

const manualGenerateEpic = async () => {
  if(!confirm('确定要手动触发今日史诗生成吗？这可能需要几十秒时间。')) return;
  isGeneratingEpic.value = true;
  try { const r = await axios.post('/archives/generate'); alert(r.data.message || '史诗生成成功！'); } 
  catch(e) { alert(e.response?.data?.error || '生成失败'); } 
  finally { isGeneratingEpic.value = false; }
};

const forceReply = async (id) => { try { const r = await axios.post('/chat/force-reply', { id }); alert(r.data.message); } catch(e) { alert(e.response?.data?.error || '激活失败'); }};

const startThinkingRotation = () => {
  let idx = 0;
  thinkingTimer = setInterval(() => { idx = (idx + 1) % thinkingTexts.length; thinkingText.value = thinkingTexts[idx]; }, 1800);
};

const runTypewriter = (text, onDone) => {
  let i = 0; displayedQuote.value = '';
  const iv = setInterval(() => {
    displayedQuote.value += text[i++];
    if (i >= text.length) { clearInterval(iv); onDone && onDone(); }
  }, 80);
};

onMounted(()=>{ fetchQuote(); fetchMessages(); initWS(); fetchUserInfo(); });
onBeforeUnmount(() => { window.removeEventListener('resize', handleResize); });

watch(currentView, (v) => {
  if(v==='archives') { fetchArchives(); }
  if(v==='my-oaths') { fetchMyOaths(); fetchMyFeedbacks(); }
  if(v==='admin') { fetchAdminFeedbacks(); }
});
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Noto+Serif+SC:wght@400;600&family=Outfit:wght@300;400;600&display=swap');

:root {
  --primary: #c0392b;
  --primary-glow: rgba(192, 57, 43, 0.4);
  --bg-dark: #050505;
  --glass: rgba(255, 255, 255, 0.03);
  --glass-border: rgba(255, 255, 255, 0.08);
  --text-main: #e0e0e0;
  --text-dim: #888;
  --font-fancy: 'Noto Serif SC', serif;
  --font-main: 'Outfit', 'Inter', sans-serif;
  --safe-bottom: env(safe-area-inset-bottom);
}

*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}

body {
  background: var(--bg-dark);
  color: var(--text-main);
  font-family: var(--font-main);
  overflow: hidden;
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

.app {
  height: 100vh;
  width: 100vw;
  display: flex;
  background: var(--bg-dark);
  position: relative;
  overflow: hidden;
}

/* 电脑端背景 */
.app::before {
  content: '';
  position: fixed;
  inset: 0;
  background-image: linear-gradient(135deg, rgba(5,5,5,0.9) 0%, rgba(5,5,5,0.4) 50%, rgba(5,5,5,0.8) 100%), 
                    url('https://xiaolongya.cn/uploads/1778566805009571718.jpg');
  background-size: cover;
  background-position: center;
  z-index: -1;
  transform: translateZ(0);
}

.main-view { 
  flex: 1; 
  display: flex; 
  flex-direction: column; 
  position: relative; 
  overflow: hidden;
  contain: size layout style;
}

.is-mobile { flex-direction: column; }
.is-mobile .main-view { padding-bottom: calc(70px + var(--safe-bottom)); }

/* 手机端背景 */
@media (max-width: 768px) {
  .app::before {
    background-image: linear-gradient(to bottom, rgba(5,5,5,0.95), rgba(5,5,5,0.7)), 
                      url('https://xiaolongya.cn/uploads/1778566827931486399.jpg');
  }
}

.t-view-enter-active, .t-view-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.t-view-enter-from { opacity: 0; transform: translateY(10px) scale(1.01); filter: blur(10px); }
.t-view-leave-to { opacity: 0; transform: translateY(-10px) scale(0.99); filter: blur(10px); }

/* 灵语签文样式 */
.fortune-overlay { 
  position: fixed; inset: 0; background: rgba(0,0,0,0.9); 
  backdrop-filter: blur(30px); z-index: 12000; 
  display: flex; align-items: center; justify-content: center; padding: 20px;
  pointer-events: auto;
}
.fortune-card { 
  width: 100%; max-width: 400px; padding: 60px 40px; text-align: center; 
  border: 1px solid rgba(255,255,255,0.05); background: rgba(10,10,10,0.9);
  box-shadow: 0 40px 100px rgba(0,0,0,0.8); border-radius: 32px;
}
.f-luck { font-family: var(--font-fancy); font-size: 3.5rem; color: #fff; margin-bottom: 8px; letter-spacing: 12px; text-shadow: 0 0 20px rgba(192,57,43,0.5); }
.f-title { font-size: 1.1rem; color: #c0392b; font-weight: bold; letter-spacing: 6px; margin-bottom: 30px; }
.f-div { height: 1px; background: linear-gradient(to right, transparent, rgba(192,57,43,0.3), transparent); margin-bottom: 30px; }
.f-interpretation { font-family: var(--font-fancy); font-size: 1.4rem; color: #eee; line-height: 1.6; margin-bottom: 40px; }

.f-grid { display: flex; flex-direction: column; gap: 16px; margin-bottom: 40px; }
.f-box { padding: 16px; border-radius: 16px; text-align: left; position: relative; overflow: hidden; }
.f-box.suit { background: rgba(192,57,43,0.08); border: 1px solid rgba(192,57,43,0.15); }
.f-box.avoid { background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.05); }

.f-box .label { font-size: 0.65rem; font-weight: bold; margin-bottom: 12px; letter-spacing: 1px; }
.f-box.suit .label { color: #c0392b; }
.f-box.avoid .label { color: #666; }
.items-list { display: flex; flex-direction: column; gap: 6px; }
.it-row { font-size: 0.95rem; color: #eee; font-weight: 500; letter-spacing: 1px; }

.btn-close-f { background: #c0392b; color: #fff; border: none; padding: 14px 50px; border-radius: 30px; font-weight: bold; cursor: pointer; transition: .3s; }
.btn-close-f:hover { transform: translateY(-3px); box-shadow: 0 10px 25px rgba(192,57,43,0.4); }

.modal-layer { position: fixed; z-index: 10000; pointer-events: none; }
.modal-layer > * { pointer-events: auto; }

.audit-mask {
  position: fixed; inset: 0; z-index: 11000; background: rgba(0,0,0,0.8);
  backdrop-filter: blur(20px); display: flex; align-items: center; justify-content: center;
}

.audit-visual { width: 180px; height: 180px; position: relative; }
.spin-ring {
  position: absolute; inset: 0; border: 2px solid rgba(192,57,43,0.1);
  border-top-color: var(--primary); border-radius: 50%;
  animation: spin 1s linear infinite;
}
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

::-webkit-scrollbar { width: 5px; height: 5px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.05); border-radius: 10px; }
::-webkit-scrollbar-thumb:hover { background: var(--primary); }
</style>
