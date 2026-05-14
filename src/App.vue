<template>
  <div id="app" :class="{'is-mobile': isMobile}">
    <div class="dragon-texture"></div>
    
    <!-- 全局导航 -->
    <GlobalNav 
      v-model="activeView" 
      :isLoggedIn="isLoggedIn" 
      :user="user" 
      :defAv="defAv"
      :isMobile="isMobile"
      @open-modal="handleOpenModal"
    />

    <!-- 主视图区域 -->
    <main class="main-content">
      <transition name="fade-view" mode="out-in">
        <component 
          :is="currentViewComponent" 
          v-bind="viewProps"
          @enter="activeView = 'chat'"
          @show-about="activeView = 'about'"
          @get-fortune="handleGetFortune"
          @send="handleSendMessage"
          @load-more="handleLoadMore"
          @del-msg="handleDeleteMessage"
          @force-reply="handleForceReply"
          @generate-image="handleGenerateImage"
          @open-modal="handleOpenModal"
          @switch-tab="handleSwitchArchiveTab"
          @post-archive="handlePostArchive"
          @show-manifesto="showManifesto = true"
          @close-manifesto="showManifesto = false"
          @update-manifesto="handleUpdateManifesto"
          @analyze-tech="handleAnalyzeTech"
          @submit="handleFeedbackSubmit"
          @reply="handleAdminReply"
          @change-page="handlePageChange"
          @manual-generate="handleManualGenerate"
          @del-fb="handleDeleteFeedback"
          @open-profile-modal="showProfileModal = true"
        />
      </transition>
    </main>

    <!-- 求签过场动画 -->
    <transition name="fade">
      <div v-if="isFortuneLoading" class="fortune-loading-overlay">
        <div class="fortune-spirit">
          <div class="spirit-core"></div>
          <div class="spirit-ring ring-1"></div>
          <div class="spirit-ring ring-2"></div>
          <div class="spirit-ring ring-3"></div>
          <div class="spirit-text-epic">
            <span class="spirit-glow-text">正在向龙主祈求灵语...</span>
            <div class="spirit-sub">因果律正在重组，请静候神谕</div>
          </div>
        </div>
      </div>
    </transition>

    <!-- 登录/注册 弹窗 -->
    <AuthModal 
      v-if="authModal" 
      :type="authModal" 
      :form="authForm" 
      :smsCooldown="smsCooldown"
      @close="authModal = null"
      @submit="handleAuth"
      @switch="(newType) => authModal = newType"
      @send-sms="handleSendSms"
    />

    <!-- 个人资料编辑 弹窗 -->
    <ProfileModal 
      v-if="showProfileModal" 
      :user="user" 
      :editForm="profileForm" 
      :defAv="defAv"
      @close="showProfileModal = false"
      @submit="handleUpdateProfile"
      @logout="handleLogout"
      @upload-avatar="handleUploadAvatar"
    />

    <!-- 求签结果 弹窗 -->
    <FortuneModal 
      v-if="showFortuneModal" 
      :fortune="currentFortune"
      @close="showFortuneModal = false"
    />

    <!-- 全局提示 (Notification) -->
    <transition name="t-toast">
      <div v-if="toast" class="global-toast" :class="toast.type">
        {{ toast.msg }}
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, watch } from 'vue';
import axios from 'axios';

// 导入组件
import GlobalNav from './components/GlobalNav.vue';
import AuthModal from './components/AuthModal.vue';
import ProfileModal from './components/ProfileModal.vue';
import FortuneModal from './components/FortuneModal.vue';
import HomeView from './views/HomeView.vue';
import ChatView from './views/ChatView.vue';
import RaisingView from './views/RaisingView.vue';
import ArchiveView from './views/ArchiveView.vue';
import FeedbackView from './views/FeedbackView.vue';
import AdminView from './views/AdminView.vue';
import AboutView from './views/AboutView.vue';
import MyOathsView from './views/MyOathsView.vue';

// 配置 Axios
axios.defaults.baseURL = '/dragon';
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// 全局状态
const isMobile = ref(window.innerWidth <= 768);
const activeView = ref('home');
const isLoggedIn = ref(false);
const user = reactive({
  id: 0, username: '', avatar: '', role: '', title: '', experience: 0, motto: ''
});
const defAv = 'https://xiaolongya.cn/uploads/1778432333617872906.jpg';
const dragonAv = 'https://xiaolongya.cn/uploads/1778649379112733278.jpg';

// 视图相关状态
const quote = reactive({ quote: '', explain: '', type: 'normal' });
const displayedQuote = ref('');
const showExplain = ref(false);
const showEnter = ref(false);
const isThinking = ref(false);
const isUpdating = ref(false);
const thinkingText = ref('正在追溯岛屿的回忆...');

const messages = ref([]);
const hasMore = ref(false);
const loadingMore = ref(false);
const chatPage = ref(1);
const chatLimit = 20;
const isChecking = ref(false);
const postCooldown = ref(0);

const archives = ref([]);
const activeArchiveTab = ref(0);
const manifesto = ref('');
const showManifesto = ref(false);

const adminFeedbacks = ref([]);
const adminTotal = ref(0);
const adminPage = ref(1);
const isGenerating = ref(false);

const myOaths = ref([]);
const myOathsTotal = ref(0);
const myOathsPage = ref(1);
const myFeedbacks = ref([]);
const userSummary = reactive({ has_dragon: false, all_tasks_done: false, magic_usage: 0 });

// 弹窗状态
const authModal = ref(null);
const authForm = reactive({ username: '', password: '', phone: '', code: '' });
const isAuthing = ref(false);
const smsCooldown = ref(0);
const showProfileModal = ref(false);
const profileForm = reactive({ nickname: '', avatar: '', motto: '' });
const showFortuneModal = ref(false);
const isFortuneLoading = ref(false);
const currentFortune = ref({});

// 提示状态
const toast = ref(null);
const showToast = (msg, type = 'info') => {
  toast.value = { msg, type };
  setTimeout(() => toast.value = null, 3000);
};

// 动态组件
const currentViewComponent = computed(() => {
  const map = {
    'home': HomeView,
    'chat': ChatView,
    'raising': RaisingView,
    'archives': ArchiveView,
    'feedback': FeedbackView,
    'admin': AdminView,
    'about': AboutView,
    'my-oaths': MyOathsView
  };
  return map[activeView.value] || HomeView;
});

// 视图 Props
const viewProps = computed(() => {
  if (activeView.value === 'home') {
    return { quote, displayedQuote: displayedQuote.value, showExplain: showExplain.value, showEnter: showEnter.value, isThinking: isThinking.value, thinkingText: thinkingText.value };
  }
  if (activeView.value === 'chat') {
    return { messages: messages.value, hasMore: hasMore.value, loadingMore: loadingMore.value, isLoggedIn: isLoggedIn.value, user, dragonAv, defAv, isChecking: isChecking.value, postCooldown: postCooldown.value, isMobile: isMobile.value, hasDragon: userSummary.has_dragon, allTasksDone: userSummary.all_tasks_done, magicUsage: userSummary.magic_usage };
  }
  if (activeView.value === 'archives') {
    return { 
      archives: archives.value, 
      isAdmin: user.role === 'admin', 
      activeTab: activeArchiveTab.value, 
      manifesto: manifesto.value,
      showManifesto: showManifesto.value 
    };
  }
  if (activeView.value === 'admin') {
    return { items: adminFeedbacks.value, total: adminTotal.value, page: adminPage.value, limit: 10, isGenerating: isGenerating.value, isUpdating: isUpdating.value };
  }
  if (activeView.value === 'my-oaths') {
    return { items: myOaths.value, total: myOathsTotal.value, page: myOathsPage.value, limit: 10, feedbacks: myFeedbacks.value, user };
  }
  return {};
});

// 监听视图切换
watch(activeView, (newView) => {
  if (newView === 'home') fetchQuote();
  if (activeView.value === 'chat') {
    fetchMessages(true);
    fetchSummary();
  }
  if (activeView.value === 'archives') fetchArchives();
  if (activeView.value === 'admin' && user.role === 'admin') fetchAdminFeedbacks();
  if (activeView.value === 'my-oaths') fetchMyOaths();
});

const fetchSummary = async () => {
  if (!isLoggedIn.value) return;
  try {
    const res = await axios.get('/raising/summary');
    Object.assign(userSummary, res.data);
  } catch (e) {}
};

// 逻辑方法
const fetchQuote = async () => {
  isThinking.value = true;
  try {
    const res = await axios.get('/quote');
    let data = res.data;
    // 如果返回的是字符串（可能包含 Markdown），尝试解析
    if (typeof data === 'string') {
      try {
        const jsonStr = data.replace(/```json|```/g, '').trim();
        data = JSON.parse(jsonStr);
      } catch (e) {
        data = { quote: data, explain: '灵力紊乱，未能解析大白话。' };
      }
    }
    Object.assign(quote, data);
    animateQuote(quote.quote || quote.content || '');
  } catch (e) {
    showToast('无法感应岛屿的呼吸', 'error');
    animateQuote('山知道我，我知道你，就已足够。');
  } finally {
    isThinking.value = false;
  }
};

const animateQuote = (text) => {
  if (!text) return;
  displayedQuote.value = '';
  showExplain.value = false;
  showEnter.value = false;
  let i = 0;
  const timer = setInterval(() => {
    if (!text[i]) {
      clearInterval(timer);
      showExplain.value = true;
      showEnter.value = true;
      return;
    }
    displayedQuote.value += text[i];
    i++;
    if (i >= text.length) {
      clearInterval(timer);
      setTimeout(() => {
        showExplain.value = true;
        showEnter.value = true;
      }, 500);
    }
  }, 100);
};

const fetchMessages = async (reset = false) => {
  try {
    let url = '/chat/list';
    if (!reset && messages.value.length > 0) {
      const oldestId = messages.value[0].id;
      url += `?before_id=${oldestId}`;
    }
    const res = await axios.get(url);
    const list = res.data.data || [];
    if (reset) {
      messages.value = list.reverse();
    } else {
      messages.value = [...list.reverse(), ...messages.value];
    }
    hasMore.value = res.data.has_more;
  } catch (e) {
    console.error('获取消息失败:', e);
  }
};

const handleSendMessage = async (content, cb) => {
  try {
    const res = await axios.post('/chat/send', { content });
    // 后端返回的是 { data: msg, ... }，WS 也会广播，但这里可以先推入以获得即时感
    if (!ws.value) {
      messages.value.push(res.data.data);
    }
    startCooldown();
    if (cb) cb(true);
  } catch (e) {
    const errMsg = e.response?.data?.error || '镌刻失败';
    if (cb) cb(false, errMsg);
  }
};

const handleLoadMore = () => {
  chatPage.value++;
  fetchMessages(false);
};

const handleDeleteMessage = async (id) => {
  try {
    await axios.delete(`/chat/${id}`);
    const msg = messages.value.find(m => m.id === id);
    if (msg) msg.is_recalled = true;
  } catch (e) {
    showToast('因果无法抹除', 'error');
  }
};

const handleForceReply = async (id) => {
  try {
    await axios.post('/chat/force-reply', { id });
    showToast('主已垂听', 'success');
  } catch (e) {
    showToast(e.response?.data?.error || '祈祷失败', 'error');
  }
};

const handleGenerateImage = async (data) => {
  isChecking.value = true;
  try {
    await axios.post('/chat/generate-image', data);
    userSummary.magic_usage++;
    startCooldown();
  } catch (e) {
    showToast('幻化失败', 'error');
  } finally {
    isChecking.value = false;
  }
};

const fetchArchives = async () => {
  try {
    const [arcRes, mfRes] = await Promise.all([
      axios.get(`/archives?type=${activeArchiveTab.value}`),
      axios.get('/archives/manifesto')
    ]);
    archives.value = arcRes.data.data || [];
    manifesto.value = mfRes.data.content || '';
  } catch (e) {}
};

const handleSwitchArchiveTab = (tab) => {
  activeArchiveTab.value = tab;
  fetchArchives();
};

const handlePostArchive = async (data) => {
  try {
    await axios.post('/archives', data);
    showToast('史诗已镌刻', 'success');
    fetchArchives();
  } catch (e) {
    showToast('镌刻失败', 'error');
  }
};

const handleAnalyzeTech = async (data, cb) => {
  try {
    const res = await axios.post('/archives/analyze', data);
    cb(res.data);
  } catch (e) {
    cb(null);
  }
};

const handleFeedbackSubmit = async (content) => {
  try {
    await axios.post('/feedback/submit', { content });
    showToast('鳞笺已投递', 'success');
    activeView.value = 'my-oaths';
    fetchMyOaths();
  } catch (e) {
    showToast('投递失败', 'error');
  }
};

const fetchAdminFeedbacks = async () => {
  try {
    const res = await axios.get(`/admin/feedback?page=${adminPage.value}&limit=10`);
    adminFeedbacks.value = res.data.data || [];
    adminTotal.value = res.data.total || 0;
  } catch (e) {}
};

const handleUpdateManifesto = async () => {
  isUpdating.value = true;
  try {
    const res = await axios.post('/admin/manifesto/update');
    showToast(res.data.message, 'success');
    fetchArchives(); // 刷新以获取可能更新的 manifesto
  } catch (e) {
    showToast(e.response?.data?.error || '重塑失败', 'error');
  } finally {
    isUpdating.value = false;
  }
};

const handleAdminReply = async (fb) => {
  try {
    await axios.post('/admin/feedback/reply', { id: fb.id, reply_content: fb.replyInput });
    showToast('回响已传达', 'success');
    fetchAdminFeedbacks();
  } catch (e) {
    showToast('传达失败', 'error');
  }
};

const handleManualGenerate = async () => {
  isGenerating.value = true;
  try {
    await axios.post('/archives/generate');
    showToast('史诗已降下', 'success');
  } catch (e) {
    showToast('生成失败', 'error');
  } finally {
    isGenerating.value = false;
  }
};

const fetchMyOaths = async () => {
  try {
    const [msgRes, fbRes] = await Promise.all([
      axios.get(`/chat/my?page=${myOathsPage.value}&limit=10`),
      axios.get('/feedback/my')
    ]);
    myOaths.value = msgRes.data.data || [];
    myOathsTotal.value = msgRes.data.total || 0;
    myFeedbacks.value = fbRes.data.data || [];
  } catch (e) {}
};

const handleDeleteFeedback = async (id) => {
  try {
    await axios.delete(`/feedback/${id}`);
    fetchMyOaths();
  } catch (e) {}
};

// Auth 相关
const checkLogin = async () => {
  const token = localStorage.getItem('token');
  if (!token) return;
  try {
    const res = await axios.get('/user/profile');
    Object.assign(user, res.data);
    isLoggedIn.value = true;
    profileForm.nickname = user.nickname || user.username;
    profileForm.avatar = user.avatar;
    profileForm.motto = user.motto;
  } catch (e) {
    localStorage.removeItem('token');
    isLoggedIn.value = false;
  }
};

const handleLogout = () => {
  localStorage.removeItem('token');
  isLoggedIn.value = false;
  user.username = '';
  user.role = '';
  showProfileModal.value = false;
  showToast('誓约已解除，后会有期', 'info');
};

const handleUploadAvatar = async (event) => {
  const file = event.target.files[0];
  if (!file) return;
  const formData = new FormData();
  formData.append('file', file);
  try {
    const res = await axios.post('/upload', formData);
    profileForm.avatar = res.data.url;
    showToast('化身幻化成功', 'success');
  } catch (e) {
    showToast('幻化失败', 'error');
  }
};

const handleOpenModal = (type) => {
  authModal.value = type;
};

const handleAuth = async () => {
  if (isAuthing.value) return;
  isAuthing.value = true;
  try {
    let url = '/auth/login';
    if (authModal.value === 'register') url = '/auth/register';
    if (authModal.value === 'forgot') url = '/auth/reset-password';

    const res = await axios.post(url, authForm);
    if (authModal.value === 'login') {
      localStorage.setItem('token', res.data.token);
      await checkLogin();
      authModal.value = null;
      showToast('欢迎回到龙屿', 'success');
    } else {
      showToast(res.data.message || '操作成功，请登录', 'success');
      authModal.value = 'login';
    }
  } catch (e) {
    showToast(e.response?.data?.error || '契约感应失败', 'error');
  } finally {
    isAuthing.value = false;
  }
};

const handleSendSms = async () => {
  const phone = authModal.value === 'register' || authModal.value === 'forgot' ? authForm.phone : authForm.username;
  if (!phone) return showToast('请输入手机号', 'warning');
  try {
    await axios.post('/auth/send-sms', { phone });
    showToast('验证码已在云端传送', 'success');
    smsCooldown.value = 60;
    const timer = setInterval(() => {
      smsCooldown.value--;
      if (smsCooldown.value <= 0) {
        clearInterval(timer);
      }
    }, 1000);
  } catch (e) {
    showToast('发送失败', 'error');
  }
};

const handleUpdateProfile = async () => {
  try {
    await axios.post('/user/profile', {
      nickname: profileForm.nickname,
      avatar: profileForm.avatar,
      motto: profileForm.motto
    });
    await checkLogin();
    showProfileModal.value = false;
    showToast('契约已重塑', 'success');
  } catch (e) {
    showToast('重塑失败', 'error');
  }
};

const startCooldown = () => {
  postCooldown.value = 10;
  const timer = setInterval(() => {
    postCooldown.value--;
    if (postCooldown.value <= 0) clearInterval(timer);
  }, 1000);
};

const handlePageChange = (p) => {
  if (activeView.value === 'admin') { adminPage.value = p; fetchAdminFeedbacks(); }
  if (activeView.value === 'my-oaths') { myOathsPage.value = p; fetchMyOaths(); }
};

const handleGetFortune = async () => {
  if (!isLoggedIn.value) {
    showToast('游侠请先签定誓约（登录）', 'warning');
    handleOpenModal('login');
    return;
  }
  isFortuneLoading.value = true;
  try {
    const res = await axios.get('/user/fortune');
    // 强制等待 2.5 秒仪式感动画
    await new Promise(resolve => setTimeout(resolve, 2500));
    
    const fortune = res.data.data;
    if (fortune) {
      currentFortune.value = fortune;
      showFortuneModal.value = true;
      showToast('签文已降临', 'success');
    }
  } catch (e) {
    showToast(e.response?.data?.error || '灵力不足，无法感应', 'error');
  } finally {
    isFortuneLoading.value = false;
  }
};

// WebSocket 逻辑
const ws = ref(null);
const initWS = () => {
  const protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
  const host = window.location.hostname === 'localhost' ? 'localhost:8888' : window.location.host;
  const wsUrl = `${protocol}//${host}/dragon/ws`;
  
  ws.value = new WebSocket(wsUrl);
  ws.value.onmessage = (e) => {
    try {
      const data = JSON.parse(e.data);
      if (data.type === 'chunk') {
        const index = messages.value.findIndex(m => m.id === data.id);
        if (index > -1) {
          messages.value[index].content += data.content;
        }
      } else if (data.id) {
        const index = messages.value.findIndex(m => m.id === data.id);
        if (index > -1) {
          messages.value[index] = { ...messages.value[index], ...data };
        } else if (activeView.value === 'chat') {
          messages.value.push(data);
        }
      }
    } catch (err) {}
  };
  ws.value.onclose = () => {
    setTimeout(initWS, 3000);
  };
};

onMounted(() => {
  checkLogin();
  fetchQuote();
  initWS();
  window.addEventListener('resize', () => {
    isMobile.value = window.innerWidth <= 768;
  });
});
</script>

<style>
/* 全局样式迁移自 style.css 或补充 */
:root {
  --text-main: #f0f0f0;
  --bg-dark: #050505;
}

body { margin: 0; background: var(--bg-dark); color: var(--text-main); font-family: 'Inter', sans-serif; overflow: hidden; height: 100vh; }

#app { display: flex; width: 100vw; height: 100vh; background: radial-gradient(circle at 50% 50%, #1a0a0a 0%, #050505 100%); }

.main-content { flex: 1; position: relative; overflow: hidden; display: flex; flex-direction: column; }

/* 视图切换动画 */
.fade-view-enter-active, .fade-view-leave-active { transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.fade-view-enter-from { opacity: 0; transform: translateY(10px); }
.fade-view-leave-to { opacity: 0; transform: translateY(-10px); }

/* 弹窗样式 */
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.8); backdrop-filter: blur(20px); z-index: 2000; display: flex; align-items: center; justify-content: center; padding: 20px; }
.modal-card { width: 100%; max-width: 440px; padding: 40px; border-radius: 32px; border: 1px solid rgba(255,255,255,0.08); background: rgba(15,15,15,0.9); box-shadow: 0 40px 100px rgba(0,0,0,0.8); }
.modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; }
.modal-header h3 { font-family: 'Noto Serif SC', serif; letter-spacing: 4px; font-size: 1.3rem; margin: 0; color: #fff; }
.btn-close { background: none; border: none; color: #444; font-size: 2rem; cursor: pointer; transition: .3s; }
.btn-close:hover { color: #c0392b; transform: rotate(90deg); }

.form-group { margin-bottom: 24px; }
.form-group label { display: block; font-size: 0.75rem; color: #555; letter-spacing: 2px; margin-bottom: 10px; font-weight: bold; }
.form-group input, .form-group textarea { width: 100%; background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1); border-radius: 12px; padding: 14px 18px; color: #fff; outline: none; transition: 0.3s; font-size: 1rem; }
.form-group input:focus { border-color: #c0392b; box-shadow: 0 0 15px rgba(192,57,43,0.2); }

.pwd-row { display: flex; gap: 12px; }
.btn-sms { background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #c0392b; border-radius: 10px; padding: 0 20px; cursor: pointer; font-size: 0.8rem; transition: .3s; }
.btn-sms:hover:not(:disabled) { background: #c0392b; color: #fff; }

.btn-auth { width: 100%; margin-top: 10px; height: 54px; font-size: 1.1rem; }
.auth-switch { margin-top: 24px; text-align: center; font-size: 0.85rem; color: #444; }
.auth-switch a { color: #c0392b; cursor: pointer; font-weight: bold; margin-left: 8px; }

/* Toast 提示 */
.global-toast { position: fixed; top: 30px; left: 50%; transform: translateX(-50%); padding: 12px 32px; border-radius: 50px; background: rgba(20,20,20,0.9); border: 1px solid rgba(255,255,255,0.1); backdrop-filter: blur(10px); z-index: 9999; color: #fff; font-size: 0.95rem; font-weight: 500; box-shadow: 0 20px 40px rgba(0,0,0,0.5); letter-spacing: 1px; }
.global-toast.error { border-color: #c0392b; color: #ff6b6b; }
.global-toast.success { border-color: #27ae60; color: #2ecc71; }

.t-toast-enter-active, .t-toast-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.t-toast-enter-from { opacity: 0; transform: translate(-50%, -20px); }
.t-toast-leave-to { opacity: 0; transform: translate(-50%, -20px); }

.t-modal-enter-active, .t-modal-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.t-modal-enter-from, .t-modal-leave-to { opacity: 0; transform: scale(0.9) translateY(20px); filter: blur(10px); }

/* 求签动画样式 */
.fortune-loading-overlay {
  position: fixed; inset: 0; z-index: 3000;
  background: radial-gradient(circle at center, #1a0a0a 0%, #000 100%);
  display: flex; align-items: center; justify-content: center;
}
.fortune-spirit { position: relative; width: 300px; height: 300px; display: flex; align-items: center; justify-content: center; flex-direction: column; }
.spirit-core { width: 12px; height: 12px; background: #c0392b; border-radius: 50%; box-shadow: 0 0 40px 10px #c0392b, 0 0 100px 20px rgba(192,57,43,0.4); z-index: 5; animation: pulse-core 1.5s infinite ease-in-out; }
.spirit-ring { position: absolute; border-radius: 50%; border: 1px solid rgba(192,57,43,0.2); }
.ring-1 { width: 60px; height: 60px; animation: spin-ring 3s linear infinite; border-top-color: #c0392b; }
.ring-2 { width: 120px; height: 120px; animation: spin-ring 5s linear infinite reverse; border-bottom-color: #c0392b; opacity: 0.6; }
.ring-3 { width: 180px; height: 180px; animation: spin-ring 8s linear infinite; border-left-color: #c0392b; opacity: 0.3; }

.spirit-text-epic { margin-top: 220px; text-align: center; }
.spirit-glow-text { font-family: 'Noto Serif SC', serif; font-size: 1.4rem; color: #fff; letter-spacing: 4px; text-shadow: 0 0 20px rgba(255,255,255,0.5); }
.spirit-sub { font-size: 0.75rem; color: #444; margin-top: 10px; letter-spacing: 2px; }

@keyframes pulse-core { 0%, 100% { transform: scale(1); opacity: 1; } 50% { transform: scale(1.5); opacity: 0.8; } }
@keyframes spin-ring { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

/* 移动端适配 */
@media (max-width: 768px) {
  .main-content { padding: 10px; padding-bottom: 90px; }
  .global-toast { top: 20px; width: 90%; left: 5%; transform: translateX(0); margin-left: 0; }
}

@font-face {
  font-family: 'Noto Serif SC';
  src: url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;700&display=swap');
}
</style>