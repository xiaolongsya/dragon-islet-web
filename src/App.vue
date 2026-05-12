<template>
  <div class="app">
    <!-- 全局导航 -->
    <GlobalNav 
      v-model="currentView" 
      :isLoggedIn="isLoggedIn" 
      :user="user" 
      :defAv="defAv"
      @open-modal="openModal" 
    />

    <main class="main-view">
      <transition name="t-view" mode="out-in">
        <!-- 视图：落地页 -->
        <HomeView 
          v-if="currentView==='home'"
          :isThinking="isThinking"
          :thinkingText="thinkingText"
          :quote="quote"
          :displayedQuote="displayedQuote"
          :showExplain="showExplain"
          :showEnter="showEnter"
          @enter="currentView='chat'"
        />

        <!-- 视图：聊天广场 -->
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
          @send="send"
          @load-more="loadMore"
          @del-msg="delMsg"
          @force-reply="forceReply"
          @generate-image="generateImage"
          @open-modal="openModal"
          @set-box-ref="val => msgBox = val"
        />

        <!-- 视图：史诗 (公共存档) -->
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

        <!-- 视图：我的历史/鳞笺 -->
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

        <!-- 视图：鳞笺投递 -->
        <FeedbackView 
          v-else-if="currentView==='feedback'"
          @submit="submitFB"
        />

        <!-- 视图：管理员 -->
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

        <!-- 视图：关于 -->
        <AboutView v-else-if="currentView==='about'" />
      </transition>
    </main>

    <!-- 弹窗 -->
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

    <!-- 审核遮罩 -->
    <div class="audit-mask" v-if="isChecking">
      <div class="audit-box pop">
        <div class="spin-ring"></div>
        <div class="ai-logo-wrap" :class="moderationStatus==='fail'?'fail':'pass'">
          <img src="https://xiaolongya.cn/uploads/1778566530694964727.jpg" class="audit-logo">
        </div>
        <div class="al">龙主正在审阅...</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, reactive, watch } from 'vue';
import axios from 'axios';

// 组件导入
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

axios.defaults.baseURL = import.meta.env.VITE_API_BASE_URL;
axios.interceptors.request.use(c => { const t = localStorage.getItem('token'); if(t) c.headers.Authorization=`Bearer ${t}`; return c; });

// 状态定义
const currentView = ref('home');
const isThinking = ref(true);
const quote = ref({ quote: '', explain: '', type: 'wisdom' });
const displayedQuote = ref('');
const showExplain = ref(false);
const showEnter = ref(false);
const thinkingTexts = ['龙主正在云端思志...', '扭动尾巴，凝谈道的精华...', '龙鲱翻涌，智慧正在凝聚...', '刚刺一裂云隙，真言将出...'];
const thinkingText = ref(thinkingTexts[0]);
let thinkingTimer = null;

const messages = ref([]); const archives = ref([]); const isChecking = ref(false);
const isLoggedIn = ref(!!localStorage.getItem('token'));
const user = ref(JSON.parse(localStorage.getItem('user')||'{}')); 
const postCooldown = ref(0); const msgBox = ref(null); const moderationStatus = ref('');
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

const defAv = `https://xiaolongya.cn/uploads/1778432333617872906.jpg`;
const dragonAv = `https://xiaolongya.cn/uploads/1778566530694964727.jpg`;

// 逻辑方法
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
    
    // 灵力同步：发送成功后请求最新用户信息
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
  messages.value = r.data.data;
  hasMore.value = r.data.has_more;
  if(messages.value.length>0) oldestID.value = messages.value[messages.value.length-1].ID;
};

const fetchArchives = async (type = 0) => {
  try {
    const r = await axios.get(`/archives?type=${type}`);
    archives.value = r.data.data;
  } catch(e) {
    console.error('获取史诗失败', e);
  }
};

const switchArchiveTab = (type) => {
  activeArchiveTab.value = type;
  fetchArchives(type);
};

const postArchive = async (data) => {
  try {
    await axios.post('/archives', data);
    alert('铸龙图谱已更新');
    fetchArchives(1);
  } catch(e) {
    alert(e.response?.data?.error || '发布失败');
  }
};

const fetchManifesto = async () => {
  try {
    const r = await axios.get('/archives/manifesto');
    manifestoContent.value = r.data.content;
  } catch(e) { alert('获取总览失败'); }
};

const analyzeTech = async (callback) => {
  try {
    const r = await axios.get('/archives/analyze');
    callback(r.data);
  } catch(e) {
    alert(e.response?.data?.error || '分析失败');
    callback(null);
  }
};

const generateImage = async ({ prompt, size }) => {
  try {
    const r = await axios.post('/chat/generate-image', { prompt, size });
    alert(r.data.message); // "龙息正在凝聚..."
  } catch(e) {
    alert(e.response?.data?.error || '幻化失败');
  }
};

const loadMore = async () => {
  if(loadingMore.value||!hasMore.value) return;
  loadingMore.value = true;
  try {
    const r = await axios.get(`/chat/list?before_id=${oldestID.value}`);
    const older = r.data.data;
    messages.value = [...messages.value, ...older];
    hasMore.value = r.data.has_more;
    if(older.length>0) oldestID.value = older[older.length-1].ID;
  } finally { loadingMore.value=false; }
};

const delMsg = async (id) => {
  if(!confirm('确定要撤回这条誓言吗？')) return;
  try { await axios.delete(`/chat/${id}`); const m = messages.value.find(x=>x.ID===id); if(m) m.is_recalled = true; } catch(e) { alert(e.response?.data?.error || '撤回失败'); }
};

const initWS = () => { 
  const ws=new WebSocket(import.meta.env.VITE_WS_URL); 
  ws.onmessage=(e)=>{ 
    const m=JSON.parse(e.data); const idx = messages.value.findIndex(x=>x.ID===m.ID);
    if(idx !== -1) messages.value[idx] = m; else messages.value.unshift(m);
  }; 
  ws.onclose=()=>setTimeout(initWS,3000); 
};

const fetchMyOaths = async () => { try { const r = await axios.get(`/chat/my?page=${myOathsPage.value}&limit=${myOathsLimit}`); myOaths.value = r.data.data; myOathsTotal.value = r.data.total; } catch(e) { console.error(e); }};
const fetchMyFeedbacks = async () => { try { const r = await axios.get('/feedback/my'); myFeedbacks.value = r.data.data; } catch {} };
const delMyMsg = async (id) => { if(!confirm('确定要抹除这条誓言吗？')) return; try { await axios.delete(`/chat/${id}`); myOaths.value = myOaths.value.filter(m => m.ID !== id); myOathsTotal.value--; } catch(e) { alert(e.response?.data?.error || '抹除失败'); }};
const delFB = async (id) => { if(!confirm('确定要抹除这份信笺吗？')) return; try { await axios.delete(`/feedback/${id}`); myFeedbacks.value = myFeedbacks.value.filter(x => x.ID !== id); } catch(e) { alert(e.response?.data?.error || '抹除失败'); }};

const submitFB = async (content) => { try { await axios.post('/feedback/submit', { content }); alert('信笺已投递'); currentView.value = 'my-oaths'; fetchMyFeedbacks(); } catch (e) { alert(e.response?.data?.error || '投递失败'); }};

const fetchAdminFeedbacks = async () => { try { const r = await axios.get(`/admin/feedback?page=${adminPage.value}&limit=${adminLimit}`); adminFeedbacks.value = r.data.data.map(fb => ({...fb, replyInput: ''})); adminTotal.value = r.data.total; } catch(e) { alert(e.response?.data?.error || '获取失败'); }};
const replyFB = async (fb) => { if(!fb.replyInput.trim()) return; try { await axios.post('/admin/feedback/reply', { id: fb.ID, content: fb.replyInput }); fb.is_replied = true; fb.reply_content = fb.replyInput; alert('回响已传达'); } catch(e) { alert(e.response?.data?.error || '回复失败'); }};

const manualGenerateEpic = async () => {
  if(!confirm('确定要手动触发今日史诗生成吗？这可能需要几十秒时间。')) return;
  isGeneratingEpic.value = true;
  try {
    const r = await axios.post('/archives/generate');
    alert(r.data.message || '史诗生成成功！');
  } catch(e) {
    alert(e.response?.data?.error || '生成失败');
  } finally {
    isGeneratingEpic.value = false;
  }
};

const forceReply = async (id) => { try { const r = await axios.post('/chat/force-reply', { id }); alert(r.data.message); } catch(e) { alert(e.response?.data?.error || '激活失败'); }};

onMounted(()=>{ fetchQuote(); fetchMessages(); initWS(); });

watch(currentView, (v) => {
  if(v==='archives') { fetchArchives(); }
  if(v==='my-oaths') { fetchMyOaths(); fetchMyFeedbacks(); }
  if(v==='admin') { fetchAdminFeedbacks(); }
});
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Noto+Serif+SC:wght@400;600&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}
.app{
  height:100vh;width:100vw;display:flex;flex-direction:row;
  color:#e0e0e0;font-family:'Inter','PingFang SC',sans-serif;overflow:hidden;
  background-color: #050505;
  background-attachment: fixed;
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  transform: translateZ(0);
  text-shadow: 0 1px 3px rgba(0,0,0,0.8); /* 增强文字识别度 */
}

/* 统一标题字体 */
h1, h2, h3, .landing-brand, .arc-title, .my-msg-header {
  font-family: 'Noto Serif SC', serif !important;
  letter-spacing: 2px;
}

/* 电脑端背景 */
@media (min-width: 769px) {
  .app { background-image: linear-gradient(135deg, rgba(5,5,5,0.92) 0%, rgba(5,5,5,0.6) 50%, rgba(5,5,5,0.8) 100%), url('https://xiaolongya.cn/uploads/1778566805009571718.jpg'); }
}

/* 手机端背景 */
@media (max-width: 768px) {
  .app { background-image: linear-gradient(to bottom, rgba(5,5,5,0.95), rgba(5,5,5,0.7)), url('https://xiaolongya.cn/uploads/1778566827931486399.jpg'); }
}

.main-view { 
  flex: 1; display: flex; flex-direction: column; position: relative; overflow: hidden; 
  background: transparent; 
  contain: layout;
}

.audit-mask{position:fixed;inset:0;z-index:9999;background:rgba(4,4,4,0.92);backdrop-filter:blur(25px);display:flex;align-items:center;justify-content:center;}
.audit-box{text-align:center;position:relative;}
.spin-ring{width:180px;height:180px;border-radius:50%;border:2px solid rgba(255,255,255,0.05);border-top-color:#c0392b;animation:spin 2s linear infinite;}
.ai-logo-wrap{position:absolute;top:35%;left:50%;transform:translate(-50%,-50%);width:110px;height:110px;border-radius:35px;overflow:hidden;border:2px solid #c0392b;box-shadow:0 0 30px rgba(192,57,43,0.3); transition: 0.3s; }
.audit-logo{width:100%;height:100%;object-fit:cover;}
.ai-logo-wrap.pass{border-color:#c0392b;box-shadow:0 0 40px rgba(192,57,43,0.5);}
.ai-logo-wrap.fail{border-color:#555;filter:grayscale(1);box-shadow:none;}
.al{margin-top:40px;color:#eee;font-size:1.2rem;letter-spacing:4px;font-family:'Noto Serif SC',serif;}
@keyframes spin{to{transform:rotate(360deg);}}

.t-view-enter-active, .t-view-leave-active { transition: opacity .3s ease, transform .3s ease; }
.t-view-enter-from { opacity: 0; transform: scale(0.98); }
.t-view-leave-to { opacity: 0; transform: scale(1.02); }
</style>
