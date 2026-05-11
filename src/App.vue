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
          @open-modal="openModal"
          @set-box-ref="val => msgBox = val"
        />

        <!-- 视图：史诗 (公共存档) -->
        <ArchiveView 
          v-else-if="currentView==='archives'"
          :archives="archives"
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
        />
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
        <div class="ai" :class="moderationStatus==='fail'?'fail':'pass'">🐉</div>
        <div class="al">龙语审阅中...</div>
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
const authForm = reactive({username:'',password:'',phone:'',code:''});
const editForm = reactive({username:'',avatar:''});
const myOaths = ref([]); const myOathsTotal = ref(0); const myOathsPage = ref(1); const myOathsLimit = 10;
const myFeedbacks = ref([]);
const adminFeedbacks = ref([]); const adminTotal = ref(0); const adminPage = ref(1); const adminLimit = 10;

const defAv = `${import.meta.env.VITE_UPLOAD_BASE_URL}/1778432333617872906.jpg`;
const dragonAv = `https://xiaolongya.cn/uploads/1778433348838960808.jpg`;

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

const openModal = (m) => { if(m==='profile'){editForm.username=user.value.username;editForm.avatar=user.value.avatar||'';} authForm.username='';authForm.password='';authForm.phone='';authForm.code=''; modal.value=m; };

const doLogin = async () => { try { const r=await axios.post('/auth/login',authForm); localStorage.setItem('token',r.data.token); localStorage.setItem('user',JSON.stringify(r.data.user)); isLoggedIn.value=true; user.value=r.data.user; modal.value=''; } catch(e){ alert(e.response?.data?.error||'登录失败'); }};
const doRegister = async () => { try { await axios.post('/auth/register',authForm); alert('注册成功'); modal.value='login'; } catch(e){ alert(e.response?.data?.error||'失败'); }};
const sendSms = async () => { if(!authForm.phone) return alert('请输入手机号'); try { await axios.post('/auth/send-sms',{phone:authForm.phone}); smsCooldown.value=60; const ti=setInterval(()=>{ if(smsCooldown.value>0)smsCooldown.value--; else clearInterval(ti); },1000); } catch(e){ alert(e.response?.data?.error||'发送失败'); }};

const updateProfile = async () => { try { await axios.post('/user/profile',{nickname:editForm.username,avatar:editForm.avatar}); user.value.username=editForm.username; if(editForm.avatar) user.value.avatar=editForm.avatar; localStorage.setItem('user',JSON.stringify(user.value)); modal.value=''; } catch(e){ alert(e.response?.data?.error||'失败'); }};
const uploadAvatar = async (e) => { const f=e.target.files[0]; if(!f) return; if(f.size>5*1024*1024) return alert('图片不能超过5MB'); const fd=new FormData(); fd.append('file',f); try { const r=await axios.post('/upload',fd); if(r.data.code===0) editForm.avatar=r.data.data.url; else alert(r.data.msg); } catch(err){ alert(err.response?.data?.msg||'上传失败'); }};
const logout = () => { localStorage.clear(); window.location.reload(); };

const send = async (content) => {
  moderationStatus.value='examining'; isChecking.value=true;
  try {
    const r=await axios.post('/chat/send',{content});
    moderationStatus.value=r.data.will_reply?'pass-interested':'pass-ignored';
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
.app{height:100vh;width:100vw;display:flex;flex-direction:row;background:#080808;color:#d0d0d0;font-family:'Inter','PingFang SC',sans-serif;overflow:hidden;}
.main-view { flex: 1; display: flex; flex-direction: column; position: relative; overflow: hidden; background: #080808; }

.audit-mask{position:fixed;inset:0;z-index:9999;background:rgba(4,4,4,0.88);backdrop-filter:blur(20px);display:flex;align-items:center;justify-content:center;}
.audit-box{background:#0d0d0d;border:1px solid rgba(180,40,40,0.35);border-radius:28px;padding:60px 80px;text-align:center;box-shadow:0 0 80px rgba(0,0,0,1);}
.spin-ring{width:56px;height:56px;border:3px solid rgba(180,40,40,0.15);border-top-color:#c0392b;border-radius:50%;animation:spin .9s linear infinite;margin:0 auto 24px;}
.ai{font-size:5.5rem;line-height:1;margin-bottom:16px;}
.pass{color:#2ecc71;filter:drop-shadow(0 0 20px rgba(46,204,113,.5));}
.fail{color:#c0392b;filter:drop-shadow(0 0 20px rgba(192,57,43,.5));}
.al{font-size:1rem;font-weight:600;letter-spacing:4px;color:#888;}
.pop{animation:pop .35s cubic-bezier(.175,.885,.32,1.275);}
@keyframes pop{from{transform:scale(0);opacity:0}to{transform:scale(1);opacity:1}}
@keyframes spin{to{transform:rotate(360deg)}}

.t-view-enter-active, .t-view-leave-active { transition: opacity .3s ease, transform .3s ease; }
.t-view-enter-from { opacity: 0; transform: scale(0.98); }
.t-view-leave-to { opacity: 0; transform: scale(1.02); }
</style>
