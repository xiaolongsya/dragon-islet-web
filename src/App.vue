<template>
  <div class="app">
    <!-- 全局侧边导航 -->
    <nav class="global-nav">
      <div class="nav-logo-wrap"><span class="nav-logo-ic">⬡</span></div>
      <div class="nav-menu">
        <div class="nav-item" :class="{active: currentView==='home'}" @click="currentView='home'">
          <span class="n-ic">⬡</span><span class="n-txt">遗境</span>
        </div>
        <div class="nav-item" :class="{active: currentView==='chat'}" @click="currentView='chat'">
          <span class="n-ic">💬</span><span class="n-txt">誓约</span>
        </div>
      </div>
      <div class="nav-user">
        <button v-if="!isLoggedIn" @click="openModal('login')" class="btn-login-sm">登</button>
        <div v-else class="nav-av" @click="openModal('profile')">
          <img :src="user.avatar||defAv">
        </div>
      </div>
    </nav>

    <!-- 主视图区 -->
    <main class="main-view">
      <!-- 审核遮罩 -->
      <transition name="t-fade">
        <div v-if="isChecking" class="audit-mask">
          <div class="audit-box">
            <template v-if="moderationStatus==='examining'"><div class="spin-ring"></div><p class="al">龙主审视中</p></template>
            <template v-else-if="moderationStatus==='pass-interested'||moderationStatus==='pass-ignored'"><div class="ai pass pop">✓</div><p class="al">{{ moderationStatus==='pass-interested'?'龙主甚感兴趣':'龙主准许通过' }}</p></template>
            <template v-else-if="moderationStatus==='fail'"><div class="ai fail pop">✕</div><p class="al">违禁，已驳回</p></template>
          </div>
        </div>
      </transition>

      <transition name="t-view" mode="out-in">
        <!-- 视图：主页 -->
        <div v-if="currentView==='home'" class="view-home" key="home">
          <div class="landing-inner">
            <div class="landing-header">
              <span class="landing-logo" :class="isThinking?'logo-thinking':''">⬡</span>
              <div class="landing-title-group">
                <div class="landing-brand">DRAGON ISLET</div>
                <div class="landing-tagline">龙屿 · 遗忘之境</div>
              </div>
            </div>
            <div class="landing-accent-line"></div>

            <!-- 思索中状态 -->
            <transition name="t-fade">
              <div v-if="isThinking" class="thinking-wrap">
                <div class="thinking-dots"><span></span><span></span><span></span></div>
                <p class="thinking-text">{{ thinkingText }}</p>
              </div>
            </transition>

            <!-- 语录内容 -->
            <transition name="t-fade">
              <div v-if="!isThinking" class="landing-quote-wrap" :class="quote.type==='secret'?'secret-glow':''">
                <p class="landing-quote">{{ displayedQuote }}<span class="cursor">|</span></p>
                <p class="landing-explain" v-if="showExplain">{{ quote.explain }}</p>
              </div>
            </transition>

            <button class="landing-enter" v-if="showEnter" @click="currentView='chat'">踏入龙屿 →</button>
            <div class="landing-hint" v-if="quote.type==='secret'&&showEnter">✦ 隐秘彩蛋 ✦</div>
          </div>
        </div>

        <!-- 视图：聊天 -->
        <div v-else-if="currentView==='chat'" class="view-chat layout" key="chat">
          <section class="chat-area">
            <div class="msg-list" ref="msgBox">
              <!-- 加载更多 -->
              <div class="load-more-wrap">
                <button v-if="hasMore" @click="loadMore" :disabled="loadingMore" class="btn-more">
                  {{ loadingMore ? '加载中...' : '查看更多' }}
                </button>
                <span v-else-if="messages.length>0" class="no-more">已到最初的誓言</span>
              </div>

              <transition-group name="t-msg">
                <div v-for="msg in messages" :key="msg.ID" class="msg-row">
                  <div class="msg-av"><img :src="msg.is_ai_reply?dragonAv:(msg.user?.avatar||defAv)"></div>
                  <div class="msg-body">
                    <div class="msg-meta">
                      <span class="msg-name" :class="msg.is_ai_reply?'name-dragon':''">{{ msg.is_ai_reply?'龙屿之主':(msg.user?.username||'游侠') }}</span>
                      <span v-if="!msg.is_ai_reply" class="itag" :class="msg.ai_interest?'itag-fire':'itag-void'">{{ msg.ai_interest?'🔥 青睐':'🌪 不屑' }}</span>
                      <span class="msg-time">{{ fmtTime(msg.CreatedAt) }}</span>
                    </div>
                    <div class="bubble" :class="msg.is_ai_reply?'b-dragon':'b-user'">{{ msg.content }}</div>
                  </div>
                </div>
              </transition-group>
            </div>

            <!-- 输入区 -->
            <div class="input-bar">
              <div v-if="!isLoggedIn" class="guest-tip" @click="openModal('login')">
                <span>✦ 唤醒龙魂后方可留下誓言</span>
                <button class="btn-p btn-sm">立即登录</button>
              </div>
              <template v-else>
                <textarea v-model="newMsg" placeholder="写下你的誓言，龙主正在聆听..." :disabled="isChecking" @keydown.ctrl.enter="send"></textarea>
                <div class="input-foot">
                  <span class="cd-txt" v-if="postCooldown>0">冷却 {{ postCooldown }}s</span>
                  <span class="hint" v-else>Ctrl+Enter 发送</span>
                  <button class="btn-p" @click="send" :disabled="isChecking||postCooldown>0||!newMsg.trim()">发布誓言</button>
                </div>
              </template>
            </div>
          </section>
        </div>
      </transition>
    </main>

    <!-- 弹窗容器 -->
    <transition name="t-modal">
      <div v-if="modal" class="modal-mask" @click.self="modal=''">
        <div class="modal-card" :class="modal==='profile'?'card-profile':''">
          <!-- 档案 -->
          <template v-if="modal==='profile'">
            <div class="pf-top">
              <label class="av-lg av-clickable">
                <img :src="editForm.avatar||user.avatar||defAv">
                <div class="av-overlay">幻化</div>
                <input type="file" hidden @change="uploadAvatar" accept="image/*">
              </label>
              <div class="pf-name">{{ user.username }}</div>
              <div class="pf-sub">游侠 · 龙屿契约者</div>
            </div>
            <div class="m-div"></div>
            <div class="form-g"><label class="f-lbl">改号</label><input class="f-inp" v-model="editForm.username" placeholder="新名号"></div>
            <button class="btn-p f-btn" @click="updateProfile">重塑契约</button>
            <button class="btn-ghost f-btn mt-s" @click="logout">归隐山林 · 登出</button>
          </template>
          <!-- 登录/注册 -->
          <template v-else>
            <div class="auth-title">{{ modal==='login'?'唤醒龙魂':'建立契约' }}</div>
            <div class="auth-div"></div>
            <div class="form-g"><label class="f-lbl">名号</label><input class="f-inp" v-model="authForm.username" placeholder="名号"></div>
            <div class="form-g"><label class="f-lbl">密语</label><input class="f-inp" v-model="authForm.password" type="password" placeholder="密语"></div>
            <template v-if="modal==='register'">
              <div class="form-g">
                <label class="f-lbl">手机验证</label>
                <div class="sms-row">
                  <input class="f-inp sms-inp" v-model="authForm.phone" placeholder="手机号">
                  <button class="btn-sms" @click="sendSms" :disabled="smsCooldown>0">{{ smsCooldown>0?smsCooldown+'s':'获取验证码' }}</button>
                </div>
              </div>
              <div class="form-g"><label class="f-lbl">验证码</label><input class="f-inp" v-model="authForm.code" placeholder="6位验证码"></div>
            </template>
            <button class="btn-p f-btn" @click="modal==='login'?doLogin():doRegister()">{{ modal==='login'?'登录':'注册' }}</button>
            <div class="auth-sw" @click="modal=modal==='login'?'register':'login'">{{ modal==='login'?'初入龙屿？建立契约':'已有契约？去登录' }}</div>
          </template>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, reactive } from 'vue';
import axios from 'axios';

axios.defaults.baseURL = import.meta.env.VITE_API_BASE_URL;
// 落地页状态
const currentView = ref('home');
const isThinking = ref(true);
const quote = ref({ quote: '', explain: '', type: 'wisdom' });
const displayedQuote = ref('');
const showExplain = ref(false);
const showEnter = ref(false);

// 思索中轮播文案
const thinkingTexts = [
  '龙主正在云端思志...',
  '扭动尾巴，凝谈道的精华...',
  '龙鲱翻涌，智慧正在凝聚...',
  '刚刺一裂云隙，真言将出...',
];
const thinkingText = ref(thinkingTexts[0]);
let thinkingTimer = null;

const startThinkingRotation = () => {
  let idx = 0;
  thinkingTimer = setInterval(() => {
    idx = (idx + 1) % thinkingTexts.length;
    thinkingText.value = thinkingTexts[idx];
  }, 1800);
};

const runTypewriter = (text, onDone) => {
  let i = 0;
  displayedQuote.value = '';
  const iv = setInterval(() => {
    displayedQuote.value += text[i++];
    if (i >= text.length) { clearInterval(iv); onDone && onDone(); }
  }, 80);
};

const fetchQuote = async () => {
  startThinkingRotation();
  try {
    const r = await axios.get('/quote');
    quote.value = r.data;
  } catch {
    quote.value = { quote: '山知道我，我知道你，就已足够。', explain: '翻译成大白话：不需要全世界都懂我，你懂就够了。', type: 'wisdom' };
  }
  clearInterval(thinkingTimer);
  // 思索状态淡出，再开始打字
  isThinking.value = false;
  setTimeout(() => {
    runTypewriter(quote.value.quote, () => {
      setTimeout(() => { showExplain.value = true; }, 600);
      setTimeout(() => { showEnter.value = true; }, 1200);
    });
  }, 400);
};


axios.interceptors.request.use(c => { const t = localStorage.getItem('token'); if(t) c.headers.Authorization=`Bearer ${t}`; return c; });

const defAv = `${import.meta.env.VITE_UPLOAD_BASE_URL}/1778432333617872906.jpg`;
const dragonAv = `${import.meta.env.VITE_UPLOAD_BASE_URL}/1778433348838960808.jpg`;

const messages = ref([]); const archives = ref([]); const newMsg = ref('');
const isChecking = ref(false); const isLoggedIn = ref(!!localStorage.getItem('token'));
const user = ref(JSON.parse(localStorage.getItem('user')||'{}')); 
const postCooldown = ref(0); const msgBox = ref(null); const moderationStatus = ref('');
const modal = ref(''); const hasMore = ref(false); const loadingMore = ref(false);
const oldestID = ref(0);
const smsCooldown = ref(0);
const authForm = reactive({username:'',password:'',phone:'',code:''});
const editForm = reactive({username:'',avatar:''});

const fmtTime = t => new Date(t).toLocaleTimeString('zh-CN',{hour:'2-digit',minute:'2-digit'});
const openModal = (m) => { if(m==='profile'){editForm.username=user.value.username;editForm.avatar=user.value.avatar||'';} authForm.username='';authForm.password='';authForm.phone='';authForm.code=''; modal.value=m; };
const sendSms = async () => { if(!authForm.phone) return alert('请输入手机号'); try { await axios.post('/auth/send-sms',{phone:authForm.phone}); smsCooldown.value=60; const ti=setInterval(()=>{ if(smsCooldown.value>0)smsCooldown.value--; else clearInterval(ti); },1000); } catch(e){ alert(e.response?.data?.error||'发送失败'); }};

const doLogin = async () => { try { const r=await axios.post('/auth/login',authForm); localStorage.setItem('token',r.data.token); localStorage.setItem('user',JSON.stringify(r.data.user)); isLoggedIn.value=true; user.value=r.data.user; modal.value=''; } catch(e){ alert(e.response?.data?.error||'登录失败'); }};
const doRegister = async () => { try { await axios.post('/auth/register',authForm); alert('注册成功'); modal.value='login'; } catch(e){ alert(e.response?.data?.error||'失败'); }};
const updateProfile = async () => { try { await axios.post('/user/profile',{nickname:editForm.username,avatar:editForm.avatar}); user.value.username=editForm.username; if(editForm.avatar) user.value.avatar=editForm.avatar; localStorage.setItem('user',JSON.stringify(user.value)); modal.value=''; } catch(e){ alert(e.response?.data?.error||'失败'); }};
const uploadAvatar = async (e) => { const f=e.target.files[0]; if(!f) return; if(f.size>5*1024*1024) return alert('图片不能超过5MB'); const fd=new FormData(); fd.append('file',f); try { const r=await axios.post('/upload',fd); if(r.data.code===0) editForm.avatar=r.data.data.url; else alert(r.data.msg); } catch(err){ alert(err.response?.data?.msg||'上传失败'); }};
const logout = () => { localStorage.clear(); window.location.reload(); };

const send = async () => {
  if(!newMsg.value.trim()||isChecking.value||postCooldown.value>0) return;
  const content=newMsg.value; moderationStatus.value='examining'; isChecking.value=true; newMsg.value='';
  try {
    const r=await axios.post('/chat/send',{content});
    moderationStatus.value=r.data.will_reply?'pass-interested':'pass-ignored';
    setTimeout(()=>{ isChecking.value=false; moderationStatus.value=''; },1400);
    postCooldown.value=60; const ti=setInterval(()=>{ if(postCooldown.value>0)postCooldown.value--; else clearInterval(ti); },1000);
  } catch(e) {
    moderationStatus.value='fail';
    setTimeout(()=>{ isChecking.value=false; moderationStatus.value=''; newMsg.value=content; alert(e.response?.data?.error||'被拦截'); },1500);
  }
};

const fetchMessages = async () => {
  const r = await axios.get('/chat/list');
  const data = r.data.data.reverse();
  messages.value = data;
  hasMore.value = r.data.has_more;
  if(data.length>0) oldestID.value = data[0].ID;
  await nextTick(); scrollBottom();
};

const loadMore = async () => {
  if(loadingMore.value||!hasMore.value) return;
  loadingMore.value = true;
  const box = msgBox.value;
  const prevH = box.scrollHeight;
  try {
    const r = await axios.get(`/chat/list?before_id=${oldestID.value}`);
    const older = r.data.data.reverse();
    messages.value = [...older, ...messages.value];
    hasMore.value = r.data.has_more;
    if(older.length>0) oldestID.value = older[0].ID;
    await nextTick();
    box.scrollTop = box.scrollHeight - prevH;
  } finally { loadingMore.value=false; }
};

const scrollBottom = () => { if(msgBox.value) msgBox.value.scrollTop=msgBox.value.scrollHeight; };
const fetchArchives = async () => { try { const r=await axios.get('/archives'); archives.value=r.data.data; } catch{}};
const initWS = () => { const wsUrl = import.meta.env.VITE_WS_URL; const ws=new WebSocket(wsUrl); ws.onmessage=(e)=>{ const m=JSON.parse(e.data); if(!messages.value.find(x=>x.ID===m.ID)){ messages.value.push(m); nextTick(scrollBottom); }}; ws.onclose=()=>setTimeout(initWS,3000); };

onMounted(()=>{ fetchQuote(); fetchMessages(); fetchArchives(); initWS(); });
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Noto+Serif+SC:wght@400;600&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}
.app{height:100vh;width:100vw;display:flex;flex-direction:row;background:#080808;color:#d0d0d0;font-family:'Inter','PingFang SC',sans-serif;overflow:hidden;}

/* --- 全局导航 --- */
.global-nav { width: 72px; flex-shrink: 0; background: #050505; border-right: 1px solid rgba(255,255,255,0.05); display: flex; flex-direction: column; align-items: center; padding: 24px 0; z-index: 100; }
.nav-logo-wrap { margin-bottom: 48px; }
.nav-logo-ic { font-size: 1.8rem; color: #c0392b; filter: drop-shadow(0 0 8px rgba(192,57,43,.5)); animation: pulse-nav 4s ease infinite;}
@keyframes pulse-nav { 0%,100%{filter:drop-shadow(0 0 8px rgba(192,57,43,.4));} 50%{filter:drop-shadow(0 0 16px rgba(192,57,43,.8));} }
.nav-menu { flex: 1; display: flex; flex-direction: column; gap: 24px; width: 100%; }
.nav-item { display: flex; flex-direction: column; align-items: center; gap: 6px; color: #555; cursor: pointer; transition: .2s; }
.nav-item:hover { color: #888; }
.nav-item.active { color: #c0392b; }
.nav-item.active .n-ic { filter: drop-shadow(0 0 8px rgba(192,57,43,.4)); }
.n-ic { font-size: 1.4rem; transition: .2s;}
.n-txt { font-size: .65rem; font-weight: 600; letter-spacing: 2px; }
.nav-user { margin-top: auto; }
.btn-login-sm { width: 40px; height: 40px; border-radius: 50%; background: rgba(192,57,43,.15); color: #c0392b; border: 1px solid rgba(192,57,43,.3); font-size: .8rem; cursor: pointer; font-weight: 600; transition: .2s; }
.btn-login-sm:hover { background: rgba(192,57,43,.3); color: #fff; }
.nav-av { width: 40px; height: 40px; border-radius: 50%; overflow: hidden; border: 2px solid rgba(255,255,255,0.1); cursor: pointer; transition: .2s; }
.nav-av:hover { border-color: #c0392b; }
.nav-av img { width: 100%; height: 100%; object-fit: cover; }

/* --- 主视图区 --- */
.main-view { flex: 1; display: flex; flex-direction: column; position: relative; overflow: hidden; background: #080808; }
.t-view-enter-active, .t-view-leave-active { transition: opacity .3s ease, transform .3s ease; }
.t-view-enter-from { opacity: 0; transform: scale(0.98); }
.t-view-leave-to { opacity: 0; transform: scale(1.02); }

/* --- 主页视图 (原落地页) --- */
.view-home { flex: 1; display: flex; align-items: center; justify-content: center; background: #050505; overflow-y: auto;}
.landing-inner{width:800px;padding:0;display:flex;flex-direction:column;}

/* 头部：logo + 品牌名横排 */
.landing-header{display:flex;align-items:center;gap:24px;margin-bottom:32px;}
.landing-logo{font-size:3.5rem;color:#c0392b;filter:drop-shadow(0 0 16px rgba(192,57,43,.5));animation:pulse 3s ease-in-out infinite;flex-shrink:0;}
.logo-thinking{animation:pulse-fast 1s ease-in-out infinite !important;}
.landing-title-group{display:flex;flex-direction:column;gap:6px;}
.landing-brand{font-size:1.1rem;font-weight:700;letter-spacing:8px;color:#555;}
.landing-tagline{font-size:.9rem;color:#444;letter-spacing:3px;}
@keyframes pulse{0%,100%{filter:drop-shadow(0 0 16px rgba(192,57,43,.4))}50%{filter:drop-shadow(0 0 36px rgba(192,57,43,.8))}}
@keyframes pulse-fast{0%,100%{filter:drop-shadow(0 0 10px rgba(192,57,43,.3));transform:scale(1)}50%{filter:drop-shadow(0 0 45px rgba(192,57,43,.9));transform:scale(1.08)}}

/* 分隔线 */
.landing-accent-line{width:100%;height:1px;background:linear-gradient(90deg,#c0392b,rgba(192,57,43,0));margin-bottom:50px;}

/* 思索中 */
.thinking-wrap{display:flex;flex-direction:column;gap:24px;padding:30px 0;}
.thinking-dots{display:flex;gap:14px;}
.thinking-dots span{width:12px;height:12px;border-radius:50%;background:#c0392b;animation:dot-bounce 1.4s ease-in-out infinite;}
.thinking-dots span:nth-child(2){animation-delay:.2s;}
.thinking-dots span:nth-child(3){animation-delay:.4s;}
@keyframes dot-bounce{0%,80%,100%{transform:scale(.6);opacity:.3}40%{transform:scale(1);opacity:1;}}
.thinking-text{font-size:1.1rem;color:#444;letter-spacing:3px;}
@keyframes text-cycle{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:none}}

/* 语录 - 左对齐，避免打字机+居中的漂移问题 */
.landing-quote-wrap{width:100%;margin-bottom:48px;}
.secret-glow .landing-quote{color:#ffb8c6;text-shadow:0 0 24px rgba(255,150,180,.2);}
.landing-quote{font-family:'Noto Serif SC',serif;font-size:3.2rem;font-weight:600;color:#f0f0f0;line-height:1.75;letter-spacing:5px;text-align:left;}
.cursor{animation:blink .8s step-end infinite;color:#c0392b;margin-left:4px;}
@keyframes blink{50%{opacity:0}}
.landing-explain{margin-top:28px;font-size:1.2rem;color:#666;line-height:2;letter-spacing:2px;animation:fade-up .5s ease both;}

/* 按钮 */
.landing-enter{align-self:flex-start;margin-top:50px;background:none;border:1px solid rgba(192,57,43,.35);color:rgba(192,57,43,.75);padding:16px 48px;border-radius:6px;cursor:pointer;font-size:1.1rem;letter-spacing:4px;transition:all .25s;font-family:inherit;animation:fade-up .4s ease both;}
.landing-enter:hover{border-color:#c0392b;color:#e0e0e0;background:rgba(192,57,43,.08);}
.landing-hint{margin-top:14px;font-size:.8rem;color:rgba(255,150,180,.35);letter-spacing:4px;animation:fade-up .4s ease both;}
@keyframes fade-up{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}
.t-landing-leave-active{transition:all .6s cubic-bezier(.7,0,.3,1);}
.t-landing-leave-to{opacity:0;transform:translateY(-50px);}

/* 审核遮罩 */
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

/* 布局 */
.layout{flex:1;display:flex;overflow:hidden;}

/* 聊天区 */
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;}
.msg-list{flex:1;overflow-y:auto;padding:24px 48px;scroll-behavior:smooth;}
.msg-list::-webkit-scrollbar{width:3px;}
.msg-list::-webkit-scrollbar-thumb{background:#1a1a1a;border-radius:3px;}

/* 加载更多 */
.load-more-wrap{text-align:center;padding:12px 0 20px;}
.btn-more{background:rgba(255,255,255,0.04);border:1px solid rgba(255,255,255,0.08);color:#666;padding:8px 24px;border-radius:20px;cursor:pointer;font-size:.78rem;transition:.2s;font-family:inherit;}
.btn-more:hover:not(:disabled){background:rgba(255,255,255,0.07);color:#aaa;}
.btn-more:disabled{opacity:.4;cursor:not-allowed;}
.no-more{font-size:.7rem;color:#333;letter-spacing:2px;}

/* 消息 */
.msg-row{display:flex;gap:14px;margin-bottom:24px;align-items:flex-start;}
.msg-av{width:40px;height:40px;border-radius:10px;overflow:hidden;border:1px solid rgba(255,255,255,0.06);flex-shrink:0;}
.msg-av img{width:40px;height:40px;object-fit:cover;display:block;}
.msg-meta{display:flex;align-items:center;gap:8px;margin-bottom:6px;flex-wrap:wrap;}
.msg-name{font-size:.8rem;font-weight:600;color:#bbb;}
.name-dragon{color:#c0392b;}
.msg-time{font-size:.65rem;color:#3a3a3a;margin-left:auto;}
.itag{font-size:.6rem;font-weight:600;padding:2px 7px;border-radius:4px;}
.itag-fire{background:rgba(200,100,0,.1);color:#c07a10;border:1px solid rgba(200,100,0,.2);}
.itag-void{background:rgba(255,255,255,.03);color:#444;border:1px solid rgba(255,255,255,.06);}
.bubble{padding:12px 16px;border-radius:4px 14px 14px 14px;line-height:1.75;font-size:.9rem;max-width:600px;}
.b-user{background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.06);color:#ccc;}
.b-dragon{background:rgba(192,57,43,.06);border:1px solid rgba(192,57,43,.15);color:#e5e5e5;border-radius:14px;}

/* 输入区 */
.input-bar{flex-shrink:0;padding:16px 48px 24px;border-top:1px solid rgba(255,255,255,0.05);}
.guest-tip{display:flex;align-items:center;justify-content:center;gap:20px;padding:16px;background:rgba(255,255,255,.02);border:1px solid rgba(255,255,255,.06);border-radius:14px;color:#555;font-size:.85rem;}
.input-bar textarea{width:100%;background:rgba(255,255,255,.03);border:1px solid rgba(255,255,255,.08);border-radius:14px;padding:14px 18px;color:#d0d0d0;font-size:.9rem;font-family:inherit;resize:none;height:84px;outline:none;transition:border-color .2s;}
.input-bar textarea:focus{border-color:rgba(192,57,43,.4);}
.input-foot{display:flex;align-items:center;justify-content:flex-end;gap:14px;margin-top:10px;}
.cd-txt{font-size:.78rem;color:#c0392b;font-weight:600;}
.hint{font-size:.7rem;color:#3a3a3a;}

/* 按钮 */
.btn-p{background:linear-gradient(135deg,#b03030,#d94040);border:none;color:#fff;padding:10px 22px;border-radius:10px;cursor:pointer;font-size:.85rem;font-weight:600;letter-spacing:.5px;transition:.2s;font-family:inherit;box-shadow:0 4px 16px rgba(180,40,40,.2);}
.btn-p:hover:not(:disabled){transform:translateY(-1px);box-shadow:0 6px 24px rgba(180,40,40,.3);}
.btn-p:disabled{opacity:.35;cursor:not-allowed;transform:none;}
.btn-sm{padding:7px 16px !important;font-size:.78rem !important;}
.btn-ghost{background:none;border:1px solid rgba(192,57,43,.2);color:rgba(192,57,43,.6);padding:10px 18px;border-radius:10px;cursor:pointer;font-size:.8rem;transition:.2s;font-family:inherit;width:100%;}
.btn-ghost:hover{border-color:#c0392b;color:#c0392b;background:rgba(192,57,43,.05);}

/* 弹窗 */
.modal-mask{position:fixed;inset:0;z-index:3000;background:rgba(0,0,0,.88);backdrop-filter:blur(16px);display:flex;align-items:center;justify-content:center;}
.modal-card{background:#0d0d0d;border:1px solid rgba(255,255,255,.07);border-radius:24px;padding:40px;width:360px;box-shadow:0 40px 100px rgba(0,0,0,.9);}
.card-profile{width:380px;}

/* 档案 */
.pf-top{display:flex;flex-direction:column;align-items:center;gap:8px;margin-bottom:24px;text-align:center;}
.av-lg{position:relative;width:90px;height:90px;border-radius:50%;overflow:hidden;border:2px solid rgba(192,57,43,.4);box-shadow:0 0 24px rgba(192,57,43,.1);cursor:pointer;transition:border-color .2s;}
.av-lg img{width:90px;height:90px;object-fit:cover;display:block;}
.av-overlay{position:absolute;inset:0;background:rgba(0,0,0,.6);color:#fff;display:flex;align-items:center;justify-content:center;font-size:.75rem;letter-spacing:2px;opacity:0;transition:.2s;}
.av-lg:hover .av-overlay{opacity:1;}
.av-lg:hover{border-color:#c0392b;box-shadow:0 0 30px rgba(192,57,43,.3);}
.pf-name{font-size:1.2rem;font-weight:700;color:#eee;}
.pf-sub{font-size:.65rem;color:#444;letter-spacing:2px;text-transform:uppercase;}
.m-div{height:1px;background:rgba(255,255,255,.05);margin:0 0 24px;}

/* 登录 */
.auth-title{font-size:1.4rem;font-weight:700;color:#fff;letter-spacing:2px;margin-bottom:6px;}
.auth-div{height:2px;width:40px;background:#c0392b;margin-bottom:28px;box-shadow:0 0 8px rgba(192,57,43,.5);}
.auth-sw{text-align:center;font-size:.75rem;color:#444;cursor:pointer;margin-top:14px;transition:.2s;}
.auth-sw:hover{color:#c0392b;}

/* 表单 */
.form-g{margin-bottom:14px;}
.f-lbl{display:block;font-size:.65rem;color:#444;letter-spacing:2px;text-transform:uppercase;margin-bottom:7px;}
.f-inp{width:100%;background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.08);border-radius:10px;padding:11px 14px;color:#d0d0d0;font-size:.88rem;font-family:inherit;outline:none;transition:border-color .2s;}
.f-inp:focus{border-color:rgba(192,57,43,.4);}
.f-btn{width:100%;padding:13px !important;font-size:.9rem !important;margin-top:6px;}
.mt-s{margin-top:10px;}

/* 短信行 */
.sms-row{display:flex;gap:8px;}
.sms-inp{flex:1;}
.btn-sms{flex-shrink:0;background:rgba(192,57,43,.12);border:1px solid rgba(192,57,43,.3);color:#c0392b;padding:0 14px;border-radius:10px;cursor:pointer;font-size:.78rem;font-family:inherit;white-space:nowrap;transition:.2s;}
.btn-sms:hover:not(:disabled){background:rgba(192,57,43,.2);}
.btn-sms:disabled{opacity:.4;cursor:not-allowed;}

/* 动画 — 最优配置 */
/* 审核遮罩：纯 opacity，GPU 直接合成，零卡顿 */
.t-fade-enter-active{transition:opacity .2s ease-out;}
.t-fade-leave-active{transition:opacity .15s ease-in;}
.t-fade-enter-from,.t-fade-leave-to{opacity:0;}

/* 弹窗蒙层：opacity only（不触发 layout/paint） */
.t-modal-enter-active{transition:opacity .22s ease-out;}
.t-modal-leave-active{transition:opacity .18s ease-in;}
.t-modal-enter-from,.t-modal-leave-to{opacity:0;}

/* 卡片：用 @keyframes 自驱动，绕开 scoped 穿透问题 */
.modal-card{animation:card-in .3s cubic-bezier(.34,1.4,.64,1) both;}
@keyframes card-in{from{transform:translateY(28px) scale(.96);opacity:0}to{transform:none;opacity:1}}

.t-msg-enter-active{transition:opacity .3s ease,transform .3s cubic-bezier(.23,1,.32,1);}
.t-msg-enter-from{opacity:0;transform:translateY(10px);}

/* --- 移动端适配 --- */
@media (max-width: 768px) {
  .app { flex-direction: column-reverse; } /* 移动端改为纵向，导航条在底部 */

  /* 底部全局导航 */
  .global-nav { width: 100%; height: 60px; flex-direction: row; padding: 0 24px; border-right: none; border-top: 1px solid rgba(255,255,255,0.05); }
  .nav-logo-wrap { display: none; }
  .nav-menu { flex-direction: row; justify-content: center; align-items: center; gap: 40px; margin: 0;}
  .nav-item { flex-direction: column; gap: 2px; }
  .nav-item .n-ic { font-size: 1.2rem; }
  .nav-user { margin-top: 0; margin-left: auto; display: flex; align-items: center;}
  .btn-login-sm, .nav-av { width: 32px; height: 32px; font-size: .7rem; }
  
  /* 落地页 */
  .view-home { align-items: center; justify-content: center; padding-top: 0; }
  .landing-inner { width: 100%; padding: 0 24px; }
  .landing-header { margin-bottom: 40px; }
  .landing-logo { font-size: 3rem; }
  .landing-brand { letter-spacing: 5px; }
  .landing-accent-line { margin-bottom: 60px; }
  .landing-quote-wrap { margin-bottom: 60px; }
  .landing-quote { font-size: 2.3rem; line-height: 1.7; letter-spacing: 3px; }
  .landing-explain { font-size: 1.1rem; margin-top: 32px; }
  .landing-enter { width: 100%; padding: 16px; text-align: center; margin-top: 40px; font-size: 1rem; }
  
  /* 聊天视图布局 */
  .view-chat { flex-direction: column; }
  
  /* 聊天区 */
  .msg-list { padding: 16px; }
  .msg-row { gap: 10px; margin-bottom: 20px; }
  .msg-av, .msg-av img { width: 34px; height: 34px; border-radius: 8px; }
  .msg-meta { gap: 6px; }
  .bubble { font-size: .85rem; padding: 10px 14px; max-width: 88%; }
  
  /* 输入区 */
  .input-bar { padding: 12px 16px 16px; }
  .guest-tip { padding: 12px; font-size: .75rem; gap: 12px; flex-direction: column; text-align: center; }
  .input-bar textarea { height: 60px; font-size: .85rem; padding: 10px 14px; }
  .hint { display: none; } /* 移动端隐藏回车提示 */
  
  /* 弹窗 */
  .modal-card, .card-profile { width: calc(100% - 32px); padding: 24px; }
  .audit-box { width: calc(100% - 32px); padding: 40px 20px; }
  .ai { font-size: 4rem; }
  .landing-accent-line { margin-bottom: 30px; }
}
</style>
