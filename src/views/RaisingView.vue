<template>
  <div class="view-raising">
    <div class="raising-container pop" v-if="loading">
      <div class="loading-state">
        <div class="loading-orb"></div>
        <span>探寻灵力波长...</span>
      </div>
    </div>

    <div class="raising-container" v-else-if="hasDragon">
      <!-- 顶部：全局状态区 -->
      <div class="top-status-bar glass-card pop">
        <div class="stat-item">
          <div class="s-header">
            <span class="s-icon">🍖</span>
            <span class="s-label">饱食度</span>
            <span class="s-num">{{ dragon.hunger }}/{{ dragon.max_hunger }}</span>
          </div>
          <div class="s-progress"><div class="s-fill hunger" :style="{width: (dragon.hunger/dragon.max_hunger)*100+'%'}"></div></div>
        </div>
        <div class="stat-item">
          <div class="s-header">
            <span class="s-icon">💖</span>
            <span class="s-label">心情值</span>
            <span class="s-num">{{ dragon.happiness }}/{{ dragon.max_happiness }}</span>
          </div>
          <div class="s-progress"><div class="s-fill happiness" :style="{width: (dragon.happiness/dragon.max_happiness)*100+'%'}"></div></div>
        </div>
        <div class="stat-item">
          <div class="s-header">
            <span class="s-icon">✨</span>
            <span class="s-label">成长值</span>
            <span class="s-num" v-if="dragon.stage < 4">{{ dragon.exp }}/{{ nextExp }}</span>
            <span class="s-num" v-else>MAX</span>
            <button v-if="dragon.stage < 4 && dragon.exp >= nextExp" class="btn-evolve pulse-gold" @click="evolve" :disabled="isActing">突破进化</button>
          </div>
          <div class="s-progress" v-if="dragon.stage < 4"><div class="s-fill exp" :style="{width: (dragon.exp/nextExp)*100+'%'}"></div></div>
          <div class="s-progress" v-else><div class="s-fill exp-max" style="width: 100%"></div></div>
        </div>
        <div class="stat-item">
          <div class="s-header">
            <span class="s-icon">🎐</span>
            <span class="s-label">今日机缘</span>
            <span class="s-num">{{ dragon.daily_event_count || 0 }}/10</span>
          </div>
          <div class="s-progress"><div class="s-fill event" :style="{width: (dragon.daily_event_count/10*100)+'%'}"></div></div>
        </div>
      </div>

      <!-- 中部：修行核心区 -->
      <div class="main-content">
        <!-- 左侧：视觉展示 -->
        <div class="visual-section pop">
          <div class="dragon-header">
            <div class="name-badge" @click="showRename=true">
              <span class="d-name">{{ dragon.name }}</span>
              <span class="d-rarity" :class="dragon.rarity">{{ rarityText }}</span>
            </div>
            <div class="header-right">
              <div class="d-stage">
                <span class="stage-tag">{{ stageText }}</span>
                <span v-if="dragon.personality" class="personality-tag">「{{ dragon.personality }}」</span>
              </div>
              <button class="btn-release-trigger" @click="showReleaseModal = true">放生</button>
            </div>
          </div>
          <!-- 全局状态标签栏 -->
          <div class="global-status-tags" v-if="dragon.statuses && dragon.statuses.length">
            <div v-for="st in dragon.statuses" :key="st.id" class="status-tag-pill" :title="st.desc + ' | ' + st.effect">
              <span class="pulse-dot"></span>
              {{ st.name }}
            </div>
          </div>

          <div class="nest-core">
            <div class="dragon-visual-wrap" :class="[{ 'egg-stage': dragon.stage === 0 }, dragon.rarity]">
              <div class="visual-glow"></div>
              <div v-if="isGenerating" class="magic-loading-wrap">
                <div class="magic-loading">
                  <div class="orb-wrap"><div class="orb"></div></div>
                  <div class="loading-text">灵力凝聚中...</div>
                </div>
              </div>
              <div v-else-if="dragon.image_url && dragon.image_url !== '[GENERATING]'" class="image-with-share">
                <img :src="dragon.image_url" class="dragon-img" @load="imgLoaded = true">
                <button class="btn-share-img" @click="shareImage" :disabled="isActing">展示真身</button>
              </div>
              <div v-else class="magic-entry" @click="generateImage">
                <div class="magic-btn-inner">
                  <span class="m-text">幻化真身</span>
                  <span class="m-sub">MANIFEST SPIRIT</span>
                </div>
              </div>
            </div>
          </div>

          <div class="quick-actions">
            <button class="act-btn" @click="feed" :disabled="isActing">投喂</button>
            <button class="act-btn" @click="play" :disabled="isActing">互动</button>
            <button class="act-btn" @click="askGuide" :disabled="isActing">求教</button>
            <button class="act-btn" @click="share" :disabled="isActing">分享</button>
          </div>
        </div>

        <!-- 右侧：动态面板 (修行任务 / 灵魂私语) -->
        <div class="tasks-section glass-card pop">
          <div class="panel-tabs">
            <div class="p-tab" :class="{active: activePanel === 'tasks'}" @click="activePanel = 'tasks'">每日修行</div>
            <div class="p-tab" :class="{active: activePanel === 'chat'}" @click="activePanel = 'chat'">灵魂私语</div>
          </div>

          <!-- 任务列表 -->
          <div v-if="activePanel === 'tasks'" class="tasks-list-wrap">
            <div class="card-header-flex">
              <span class="rarity-bonus" v-if="dragon.rarity !== 'common'">{{ dragon.rarity === 'epic' ? '5.0x' : '2.5x' }} 奖励</span>
            </div>
            <div class="tasks-list">
              <div v-if="tasks.length===0" class="task-empty">暂无修行任务</div>
              <div v-for="task in tasks" :key="task.id" class="task-item" :class="{ 'is-completed': task.progress >= task.max_progress, 'is-claimed': task.is_claimed }">
                <div class="t-info">
                  <div class="t-name">{{ taskLabels[task.task_type || task.TaskType] || '修行任务' }}</div>
                  <div class="t-progress-text">{{ task.progress ?? 0 }}/{{ task.max_progress || task.MaxProgress || 1 }}</div>
                </div>
                <div class="t-action">
                  <button v-if="task.progress >= task.max_progress && !task.is_claimed" class="btn-claim-reward" @click="claimReward(task.id)">领赏</button>
                  <span v-else-if="task.is_claimed" class="t-status-done">已圆满</span>
                  <div v-else class="t-mini-bar"><div class="t-mini-fill" :style="{width: (task.progress/task.max_progress*100)+'%'}"></div></div>
                </div>
              </div>
            </div>
          </div>

          <!-- 私聊界面 -->
          <div v-else class="dragon-chat-wrap">
            <div class="soul-status">
              <div class="soul-personality">
                <span class="s-p-label">灵魂特质:</span>
                <span class="s-p-val">{{ dragon.soul || '纯净无瑕，待你塑魂' }}</span>
              </div>
              <div class="soul-memory" v-if="dragon.memory">
                <span class="s-p-label">深层记忆:</span>
                <span class="s-p-val">{{ dragon.memory }}</span>
              </div>
            </div>

            <div class="d-chat-messages" ref="chatScroll">
              <div v-if="hasMoreChat" class="chat-load-more" @click="loadMoreChat">查看更多往昔回顾...</div>
              <div v-for="(msg, idx) in dragonChat" :key="idx" class="d-msg" :class="msg.role">
                <div class="d-msg-bubble" :class="{ thinking: !msg.content && msg.role === 'dragon' }">
                  {{ msg.content || (msg.role === 'dragon' ? '...' : '') }}
                </div>
              </div>
            </div>

            <div class="d-chat-input-row">
              <input v-model="chatInput" @keyup.enter="sendChat" placeholder="倾听它的心声..." :disabled="isChatting">
              <button @click="sendChat" :disabled="isChatting || !chatInput.trim()">唤起</button>
            </div>
          </div>
        </div>
      </div>

      <!-- 底部：囊中之物 -->
      <div class="bottom-section glass-card pop">
        <div class="card-header">龙之囊中物</div>
        <div class="items-grid-scroll">
          <div v-if="items.length===0" class="item-empty">空空如也</div>
          <div v-for="item in items" :key="item.type" class="item-slot" :class="{ 'is-selected': selectedItem === item.type }" @click="selectedItem = item.type === selectedItem ? null : item.type">
            <div class="item-icon">{{ getItemIcon(item.type) }}</div>
            <div class="item-count">x{{ item.count }}</div>
            <div class="item-name-tag">{{ getItemName(item.type) }}</div>
            
            <!-- 使用遮罩层 -->
            <transition name="fade">
              <div v-if="selectedItem === item.type && isUsable(item.type)" class="use-overlay">
                <button class="btn-use-inner" @click.stop="useItem(item.type)">使用</button>
              </div>
            </transition>
          </div>
        </div>
      </div>

      <!-- 奇遇弹窗 -->
      <transition name="fade">
        <div v-if="randomEvent" class="event-overlay" @click="randomEvent = null">
          <div class="event-card" @click.stop>
            <div class="event-title">◈ 奇遇时刻 ◈</div>
            <div class="event-story">{{ randomEvent }}</div>
            <button class="event-close" @click="randomEvent = null">铭记于心</button>
          </div>
        </div>
      </transition>
    </div>

    <!-- 无龙状态 -->
    <div class="raising-container empty-state" v-else>
      <div class="empty-card glass-card pop">
        <div class="empty-icon">🥚</div>
        <h2 class="empty-title">尚未契约龙嗣</h2>
        <p class="empty-desc">在这片神秘的岛屿上，只有留下真挚誓言的游侠，才能获得龙主的眷顾。</p>
        
        <div class="guide-steps">
          <div class="step">
            <span class="step-num">1</span>
            <span class="step-txt">前往「誓约广场」留下你的誓言</span>
          </div>
          <div class="step">
            <span class="step-num">2</span>
            <span class="step-txt">诚心祈求龙主赐予龙蛋</span>
          </div>
          <div class="step">
            <span class="step-num">3</span>
            <span class="step-txt">一旦获得龙蛋，即可在此开始修行</span>
          </div>
        </div>

        <button class="btn-goto-plaza pulse-gold" @click="$emit('switch-view', 'chat')">前往广场</button>
      </div>
    </div>

    <!-- 改名弹窗 -->
    <transition name="pop">
      <div class="rename-modal-overlay" v-if="showRename" @click.self="showRename=false">
        <div class="rename-card glass-card pop">
          <h3>重塑真名</h3>
          <input v-model="newName" placeholder="输入新的名字..." maxlength="10">
          <div class="modal-btns">
            <button class="btn-cancel" @click="showRename=false">取消</button>
            <button class="btn-confirm" @click="renameDragon" :disabled="!newName.trim()">确认</button>
          </div>
        </div>
      </div>
    </transition>
    <!-- 放生弹窗 -->
    <transition name="pop">
      <div class="rename-modal-overlay" v-if="showReleaseModal" @click.self="showReleaseModal=false">
        <div class="rename-card glass-card pop">
          <div class="release-warning">
            <h3>解除契约</h3>
            <p>放生后，龙宝宝将回归大自然，所有的修行成果、记忆与灵魂都将消散。此操作不可逆！</p>
          </div>
          <input v-model="releasePassword" type="password" placeholder="请输入登录密语以确认..." class="release-pw-input">
          <div class="modal-btns">
            <button class="btn-cancel" @click="showReleaseModal=false">我再想想</button>
            <button class="btn-confirm btn-danger" @click="releaseDragon" :disabled="!releasePassword.trim() || isActing">确认放生</button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, onBeforeUnmount } from 'vue';
import axios from 'axios';

const taskLabels = { 
  'sign_in': '每日签到', 
  'chat': '广场传音', 
  'generate': '灵力显像', 
  'share': '真身展示',
  'feed': '每日投喂',
  'play': '灵力互动',
  'fortune': '每日求签'
};

const loading = ref(true);
const hasDragon = ref(false);
const dragon = ref({});
const items = ref([]);
const tasks = ref([]);
const rarityMult = ref(1);

const randomEvent = ref(null);
const selectedItem = ref(null);
const activePanel = ref('tasks');
const dragonChat = ref([]);
const chatInput = ref('');
const isChatting = ref(false);
const chatPage = ref(1);
const hasMoreChat = ref(true);
const chatScroll = ref(null);

const getItemIcon = (type) => {
  const icons = { 'food': '◆', 'exp_pill': '◈', 'sacrifice_stone': '◇' };
  return icons[type] || '▣';
};

const getItemName = (type) => {
  const names = { 'food': '龙粮', 'exp_pill': '龙髓丹', 'sacrifice_stone': '献祭之石' };
  return names[type] || '神秘物品';
};

const isUsable = (type) => ['exp_pill', 'sacrifice_stone'].includes(type);

const useItem = async (type) => {
  if (isActing.value) return;
  isActing.value = true;
  try {
    const r = await axios.post('/raising/use-item', { type });
    alert(r.data.message);
    await fetchStatus();
    selectedItem.value = null;
  } catch(e) { alert(e.response?.data?.error || '使用失败'); }
  finally { isActing.value = false; }
};

const isActing = ref(false);
const showRename = ref(false);
const showReleaseModal = ref(false);
const releasePassword = ref('');
const newName = ref('');
const isGenerating = ref(false);
const imgLoaded = ref(false);

const rarityText = computed(() => {
  const map = { 'common': '凡俗', 'rare': '珍稀', 'epic': '史诗' };
  return map[dragon.value.rarity] || '未知';
});

const stageText = computed(() => {
  const map = ['龙蛋', '幼龙', '青年龙', '壮年龙', '真龙'];
  return map[dragon.value.stage] || '幻化中';
});

const nextExp = computed(() => {
  if (dragon.value.stage >= 4) return 0; // 无上限
  return (dragon.value.stage + 1) * 200;
});

const evolve = async () => {
  if (isActing.value) return;
  if (!confirm(`确定要消耗灵力进行突破进化吗？\n进化后将步入新阶段，上限大幅提升！`)) return;
  isActing.value = true;
  try {
    const r = await axios.post('/raising/evolve');
    alert(r.data.message);
    await fetchStatus();
  } catch(e) { alert(e.response?.data?.error || '进化失败'); }
  finally { isActing.value = false; }
};

const fetchStatus = async () => {
  try {
    const r = await axios.get('/raising/status');
    hasDragon.value = r.data.has_dragon;
    dragon.value = r.data.dragon || {};
    items.value = r.data.items || [];
    newName.value = dragon.value.name || '';
    if (dragon.value.image_url === '[GENERATING]') {
      isGenerating.value = true;
      startPolling();
    } else {
      isGenerating.value = false;
    }
    if (hasDragon.value) fetchChatHistory();
  } catch(e) { console.error(e); }
  finally { loading.value = false; }
};

const fetchChatHistory = async (page = 1) => {
  try {
    const r = await axios.get(`/raising/chat?page=${page}&pageSize=20`);
    const newMsgs = r.data || [];
    if (page === 1) {
      dragonChat.value = newMsgs;
      scrollToBottom();
    } else {
      if (newMsgs.length > 0) {
        dragonChat.value = [...newMsgs, ...dragonChat.value];
      }
    }
    hasMoreChat.value = newMsgs.length === 20;
    chatPage.value = page;
  } catch {}
};

const loadMoreChat = () => {
  fetchChatHistory(chatPage.value + 1);
};

const sendChat = async () => {
  if (isChatting.value || !chatInput.value.trim()) return;
  const content = chatInput.value;
  chatInput.value = '';
  isChatting.value = true;
  
  dragonChat.value.push({ role: 'user', content });
  const replyIndex = dragonChat.value.length;
  dragonChat.value.push({ role: 'dragon', content: '' });
  scrollToBottom();

  try {
    const response = await fetch('/dragon/raising/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + localStorage.getItem('token')
      },
      body: JSON.stringify({ content })
    });

    if (!response.ok) throw new Error('灵力波动异常');

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let currentEvent = 'message';
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      
      const chunk = decoder.decode(value);
      const lines = chunk.split('\n');
      for (const line of lines) {
        if (line.startsWith('event:')) {
          currentEvent = line.replace('event:', '').trim();
        } else if (line.startsWith('data:')) {
          const data = line.replace('data:', '').trim();
          if (currentEvent === 'message') {
            dragonChat.value[replyIndex].content += data;
            scrollToBottom();
          } else if (currentEvent === 'event') {
            randomEvent.value = data;
            fetchStatus(); // 触发奇遇后刷新状态以显示奖励
          }
        } else if (line === '') {
          currentEvent = 'message';
        }
      }
    }
    
    if (dragonChat.value.length % 10 === 0) fetchStatus();
  } catch (e) {
    alert('对话失败：' + e.message);
  } finally {
    isChatting.value = false;
  }
};

const scrollToBottom = () => {
  setTimeout(() => {
    if (chatScroll.value) {
      chatScroll.value.scrollTop = chatScroll.value.scrollHeight;
    }
  }, 100);
};

watch(activePanel, (newVal) => {
  if (newVal === 'chat') {
    scrollToBottom();
  }
});

const fetchTasks = async () => {
  try {
    const r = await axios.get('/raising/tasks');
    tasks.value = r.data.data ? r.data.data : r.data;
  } catch {}
};

const feed = async () => {
  if (isActing.value) return;
  isActing.value = true;
  try {
    const r = await axios.post('/raising/feed');
    if (r.data.event) randomEvent.value = r.data.event;
    await fetchStatus();
  } catch(e) { alert(e.response?.data?.error || '喂食失败'); }
  finally { isActing.value = false; }
};

const play = async () => {
  if (isActing.value) return;
  isActing.value = true;
  try {
    const r = await axios.post('/raising/play');
    if (r.data.event) randomEvent.value = r.data.event;
    await fetchStatus();
  } catch(e) { alert(e.response?.data?.error || '陪玩失败'); }
  finally { isActing.value = false; }
};

const renameDragon = async () => {
  try {
    await axios.post('/raising/rename', { name: newName.value });
    dragon.value.name = newName.value;
    showRename.value = false;
  } catch(e) { alert(e.response?.data?.error || '失败'); }
};

const generateImage = async () => {
  if (isGenerating.value) return;
  try {
    await axios.post('/raising/generate-image');
    isGenerating.value = true;
    startPolling();
  } catch(e) { alert(e.response?.data?.error || '失败'); }
};

const shareImage = async () => {
  if (isActing.value) return;
  isActing.value = true;
  try {
    await axios.post('/raising/share');
    alert('真身已分享至誓约广场');
    await fetchTasks();
  } catch(e) { alert(e.response?.data?.error || '分享失败'); }
  finally { isActing.value = false; }
};

const releaseDragon = async () => {
  if (isActing.value || !releasePassword.value.trim()) return;
  if (!confirm('最后的确认：你真的要让它离开吗？')) return;
  
  isActing.value = true;
  try {
    const r = await axios.post('/raising/release', { password: releasePassword.value });
    alert(r.data.message);
    showReleaseModal.value = false;
    releasePassword.value = '';
    await fetchStatus();
  } catch(e) {
    alert(e.response?.data?.error || '身份校验失败');
  } finally {
    isActing.value = false;
  }
};

const claimReward = async (taskId) => {
  try {
    const r = await axios.post('/raising/claim-reward', { id: taskId });
    alert(r.data.message);
    fetchStatus();
    fetchTasks();
  } catch(e) { alert(e.response?.data?.error || '失败'); }
};

const askGuide = () => alert('龙主低语：勤加修行，终成正果。');
const share = () => alert('请点击【分享真身】按钮，向众生展示你的龙宝宝。');

let pollTimer = null;
const startPolling = () => {
  if (pollTimer) clearInterval(pollTimer);
  pollTimer = setInterval(async () => {
    try {
      const r = await axios.get('/raising/status');
      if (r.data.dragon?.image_url !== '[GENERATING]') {
        dragon.value.image_url = r.data.dragon.image_url;
        isGenerating.value = false;
        clearInterval(pollTimer);
      }
    } catch {}
  }, 10000);
};


onMounted(() => {
  fetchStatus();
  fetchTasks();
});
onBeforeUnmount(() => clearInterval(pollTimer));
</script>

<style scoped>
.view-raising { 
  flex: 1; display: flex; flex-direction: column; min-height: 100vh; 
  position: relative; background: transparent;
}

.raising-container { 
  max-width: 1200px; width: 100%; margin: 0 auto; 
  padding: 20px; display: flex; flex-direction: column; gap: 20px;
}

.glass-card { 
  background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08); 
  border-radius: 24px; padding: 24px; backdrop-filter: blur(15px); 
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
}

/* 顶部：状态条 */
.top-status-bar { 
  display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; padding: 20px 25px; 
}
.stat-item { flex: 1; min-width: 0; }
.s-header { display: flex; align-items: center; gap: 6px; margin-bottom: 8px; font-weight: bold; font-size: 0.85rem; }
.s-icon { font-size: 1rem; }
.s-label { color: rgba(255,255,255,0.7); flex: 1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.s-num { font-size: 0.7rem; color: #fff; font-family: 'Outfit', sans-serif; opacity: 0.5; }
.s-progress { height: 6px; background: rgba(255,255,255,0.05); border-radius: 10px; overflow: hidden; }
.s-fill { height: 100%; transition: 1s cubic-bezier(0.23, 1, 0.32, 1); border-radius: 10px; }
.hunger { background: linear-gradient(90deg, #e67e22, #f1c40f); box-shadow: 0 0 10px rgba(230,126,34,0.3); }
.happiness { background: linear-gradient(90deg, #e91e63, #ff4081); box-shadow: 0 0 10px rgba(233,30,99,0.3); }
.exp { background: linear-gradient(90deg, #2ecc71, #00bcd4); box-shadow: 0 0 10px rgba(46,204,113,0.3); }
.event { background: linear-gradient(90deg, #9b59b6, #8e44ad); box-shadow: 0 0 10px rgba(155,89,182,0.3); }

.btn-evolve {
  background: linear-gradient(135deg, #ffd700, #ff8c00); color: #000; border: none;
  padding: 4px 12px; border-radius: 8px; font-weight: 900; font-size: 0.75rem;
  margin-left: 15px; cursor: pointer; transition: 0.3s;
  box-shadow: 0 0 15px rgba(255, 215, 0, 0.4);
}
.pulse-gold { animation: pulse-gold 2s infinite; }
@keyframes pulse-gold { 0%,100% { transform: scale(1); box-shadow: 0 0 15px #ffd700; } 50% { transform: scale(1.05); box-shadow: 0 0 30px #ffd700; } }

.exp-max { background: linear-gradient(90deg, #c0392b, #ffd700, #c0392b); background-size: 200% 100%; animation: magic-flow 3s linear infinite; }
@keyframes magic-flow { 0% { background-position: 0% 50%; } 100% { background-position: 200% 50%; } }

/* 中部：主内容区 */
.main-content { 
	display: grid; grid-template-columns: 1fr 400px; gap: 20px;
}

.visual-section { 
  display: flex; flex-direction: column; padding: 30px;
}
.dragon-header { display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 20px; }
.d-name { font-family: 'Noto Serif SC', serif; font-size: 2.5rem; color: #fff; }
.d-rarity { font-size: 0.7rem; padding: 2px 10px; border-radius: 6px; font-weight: 900; letter-spacing: 1px; }
.d-rarity.epic { background: #c0392b; color: #fff; box-shadow: 0 0 15px rgba(192,57,43,0.5); }
.d-stage { color: #666; font-size: 0.9rem; display: flex; align-items: center; gap: 8px; margin-bottom: 4px; }
.personality-tag { color: #ffd700; font-style: italic; font-size: 0.8rem; font-weight: bold; }
.header-right { text-align: right; }
.btn-release-trigger { background: none; border: 1px solid rgba(255,255,255,0.1); color: #555; font-size: 0.7rem; padding: 2px 10px; border-radius: 4px; cursor: pointer; transition: 0.3s; }
.btn-release-trigger:hover { color: #c0392b; border-color: #c0392b; }

.nest-core { display: flex; justify-content: center; align-items: center; padding: 40px 0; }
.dragon-visual-wrap { 
  width: 100%; max-width: 450px; aspect-ratio: 1; border-radius: 40px; 
  overflow: hidden; border: 1px solid rgba(255,255,255,0.1); background: #000;
  position: relative; box-shadow: 0 40px 80px rgba(0,0,0,0.6);
}

/* 灵力凝聚动画 */
.magic-loading-wrap {
  position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;
  background: rgba(0,0,0,0.8); z-index: 100;
}
.magic-loading { text-align: center; }
.orb-wrap { 
  width: 120px; height: 120px; margin: 0 auto 30px; position: relative;
  border: 2px solid rgba(192,57,43,0.2); border-radius: 50%;
  padding: 10px; animation: rotate-gear 4s linear infinite;
}
.orb {
  width: 100%; height: 100%; background: radial-gradient(circle, #ff3c00 0%, #c0392b 50%, transparent 100%);
  border-radius: 50%; box-shadow: 0 0 40px #c0392b;
  animation: orb-pulse 1.5s ease-in-out infinite alternate;
}
.loading-text { 
  color: #c0392b; font-size: 1.1rem; font-weight: bold; letter-spacing: 4px;
  animation: text-blink 1s infinite alternate;
}

@keyframes rotate-gear { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
@keyframes orb-pulse { from { transform: scale(0.8); opacity: 0.6; } to { transform: scale(1.1); opacity: 1; } }
@keyframes text-blink { from { opacity: 0.3; text-shadow: 0 0 5px #c0392b; } to { opacity: 1; text-shadow: 0 0 20px #c0392b; } }

/* 幻化入口美化 */
.magic-entry { 
  position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;
  cursor: pointer; background: radial-gradient(circle at center, rgba(192,57,43,0.1) 0%, transparent 80%);
  transition: 0.5s; overflow: hidden;
}
.magic-entry::before {
  content: '✦'; position: absolute; font-size: 15rem; color: rgba(255,215,0,0.03);
  animation: spin-slow 30s linear infinite;
}
.magic-btn-inner {
  position: relative; z-index: 2; padding: 40px 60px;
  background: rgba(0,0,0,0.6); backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 215, 0, 0.2); border-radius: 40px;
  box-shadow: 0 0 50px rgba(0,0,0,0.8), inset 0 0 20px rgba(255,215,0,0.1);
  transition: 0.6s cubic-bezier(0.23, 1, 0.32, 1);
  display: flex; flex-direction: column; align-items: center; gap: 4px;
}
.magic-entry:hover .magic-btn-inner {
  transform: translateY(-10px) scale(1.05);
  border-color: #ffd700;
  box-shadow: 0 20px 60px rgba(0,0,0,0.9), 0 0 30px rgba(192,57,43,0.4);
}
.m-text {
  font-family: 'Noto Serif SC', serif; font-size: 1.8rem; color: #ffd700;
  letter-spacing: 12px; font-weight: 900; 
  text-shadow: 0 0 15px rgba(255,215,0,0.4);
  background: linear-gradient(to right, #fff, #ffd700, #fff);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-size: 200% auto; animation: shine 3s linear infinite;
}
.m-sub {
  font-size: 0.75rem; color: rgba(255,255,255,0.4); text-transform: uppercase;
  letter-spacing: 4px; margin-top: 5px; font-weight: bold;
}

@keyframes shine { to { background-position: 200% center; } }
@keyframes spin-slow { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

.dragon-img { width: 100%; height: 100%; object-fit: cover; }
.btn-share-img { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.5); padding: 8px 20px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.2); color: #fff; font-size: 0.8rem; cursor: pointer; }

.quick-actions { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
.act-btn { 
  background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.1); 
  padding: 12px; border-radius: 16px; color: #fff; cursor: pointer; transition: 0.3s;
  font-size: 0.9rem; font-weight: bold;
}
.act-btn:hover:not(:disabled) { background: #c0392b; border-color: #c0392b; transform: translateY(-3px); }

/* 任务列表 */
/* 面板切换 */
.panel-tabs { display: flex; gap: 20px; border-bottom: 1px solid rgba(255,255,255,0.05); margin-bottom: 20px; }
.p-tab { padding: 10px 0; font-size: 0.9rem; color: #555; cursor: pointer; transition: 0.3s; position: relative; }
.p-tab.active { color: #fff; font-weight: bold; }
.p-tab.active::after { content: ''; position: absolute; bottom: -1px; left: 0; width: 100%; height: 2px; background: #c0392b; }

.tasks-section { display: flex; flex-direction: column; height: 600px; }
.tasks-list-wrap { flex: 1; display: flex; flex-direction: column; }
.tasks-list { flex: 1; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }

/* 私聊界面 */
.dragon-chat-wrap { flex: 1; display: flex; flex-direction: column; min-height: 0; }
.soul-status { background: rgba(0,0,0,0.3); padding: 12px; border-radius: 12px; margin-bottom: 15px; font-size: 0.75rem; border: 1px solid rgba(255,255,255,0.05); }
.soul-personality, .soul-memory { margin-bottom: 5px; display: flex; gap: 8px; }
.s-p-label { color: #c0392b; font-weight: bold; flex-shrink: 0; }
.s-p-val { color: #888; font-style: italic; }

.d-chat-messages { flex: 1; overflow-y: auto; padding: 10px; display: flex; flex-direction: column; gap: 15px; margin-bottom: 15px; }
.d-chat-messages::-webkit-scrollbar { width: 4px; }
.d-chat-messages::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.05); border-radius: 10px; }

.d-msg { display: flex; max-width: 85%; }
.d-msg.user { align-self: flex-end; }
.d-msg.dragon { align-self: flex-start; }
.d-msg-bubble { padding: 10px 15px; border-radius: 16px; font-size: 0.9rem; line-height: 1.5; }
.d-msg.user .d-msg-bubble { background: #c0392b; color: #fff; border-bottom-right-radius: 4px; }
.d-msg.dragon .d-msg-bubble { background: rgba(255,255,255,0.05); color: #ccc; border: 1px solid rgba(255,255,255,0.1); border-bottom-left-radius: 4px; }

.d-msg-bubble.thinking { animation: thinking 1s infinite alternate; }
@keyframes thinking { from { opacity: 0.3; } to { opacity: 1; } }

.global-status-tags { 
  display: flex; flex-wrap: wrap; gap: 10px; margin: 15px 0; 
  padding: 10px; background: rgba(255,255,255,0.02); border-radius: 12px;
}
.status-tag-pill {
  background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3);
  color: #ff4d4d; font-size: 0.75rem; padding: 4px 12px; border-radius: 20px;
  display: flex; align-items: center; gap: 6px; cursor: help;
  transition: 0.3s;
}
.status-tag-pill:hover { background: rgba(192,57,43,0.2); transform: scale(1.05); }
.pulse-dot { width: 6px; height: 6px; background: #ff4d4d; border-radius: 50%; box-shadow: 0 0 10px #ff4d4d; animation: dot-pulse 1s infinite alternate; }
@keyframes dot-pulse { from { transform: scale(0.8); opacity: 0.5; } to { transform: scale(1.2); opacity: 1; } }

.d-chat-input-row { display: flex; gap: 10px; }

.chat-load-more {
  text-align: center; font-size: 0.7rem; color: #555; padding: 10px;
  cursor: pointer; transition: 0.3s;
}
.chat-load-more:hover { color: #c0392b; }
.d-chat-input-row input { flex: 1; background: rgba(0,0,0,0.3); border: 1px solid rgba(255,255,255,0.1); border-radius: 10px; padding: 10px 15px; color: #fff; outline: none; }
.d-chat-input-row button { background: #c0392b; border: none; color: #fff; padding: 0 20px; border-radius: 10px; font-weight: bold; cursor: pointer; }
.d-chat-input-row button:disabled { opacity: 0.5; }
.task-item { 
  background: rgba(0,0,0,0.2); padding: 15px; border-radius: 16px; 
  display: flex; justify-content: space-between; align-items: center;
}
.t-name { font-size: 0.9rem; color: #fff; margin-bottom: 4px; }
.t-progress-text { font-size: 0.75rem; color: #666; }
.btn-claim-reward { background: #c0392b; border: none; color: #fff; padding: 5px 15px; border-radius: 8px; font-weight: bold; cursor: pointer; }

/* 底部：背包 */
.bottom-section { padding: 20px; }
.items-grid-scroll { 
  display: flex; gap: 15px; overflow-x: auto; padding: 10px 0;
}
.items-grid-scroll::-webkit-scrollbar { height: 4px; }
.items-grid-scroll::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 2px; }

.item-slot { 
  min-width: 80px; height: 80px; background: rgba(255,255,255,0.03); 
  border-radius: 16px; border: 1px solid rgba(255,255,255,0.1);
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  position: relative; cursor: pointer; transition: 0.3s;
}
.item-slot:hover { background: rgba(255,255,255,0.08); transform: translateY(-5px); }
.item-slot.is-selected { 
  border-color: #c0392b; background: rgba(192,57,43,0.1); 
  box-shadow: 0 0 15px rgba(192,57,43,0.3);
}
.item-icon { font-size: 1.8rem; }
.item-count { position: absolute; top: 5px; right: 8px; font-size: 0.7rem; color: #c0392b; font-weight: 900; }
.item-name-tag { font-size: 0.6rem; color: #555; }

.use-overlay {
  position: absolute; inset: 0; background: rgba(192,57,43,0.9);
  display: flex; align-items: center; justify-content: center;
  border-radius: 16px; z-index: 10; animation: fade-in 0.2s;
}
.btn-use-inner { 
  background: #fff; color: #c0392b; border: none; padding: 4px 10px; 
  border-radius: 8px; font-weight: 900; font-size: 0.75rem; cursor: pointer;
  box-shadow: 0 4px 10px rgba(0,0,0,0.3);
}
.btn-use-inner:hover { transform: scale(1.1); }
@keyframes fade-in { from { opacity: 0; } to { opacity: 1; } }

/* 响应式适配 */
@media (max-width: 1000px) {
  .main-content { grid-template-columns: 1fr; }
  .tasks-section { height: 300px; }
  .top-status-bar { grid-template-columns: repeat(2, 1fr); gap: 15px; }
}

@media (max-width: 600px) {
  .raising-container { padding: 15px 15px 100px; gap: 15px; }
  .d-name { font-size: 1.8rem; }
  .nest-core { padding: 10px 0; }
  .dragon-visual-wrap { border-radius: 24px; }
  .quick-actions { grid-template-columns: repeat(2, 1fr); gap: 10px; }
  .item-slot { min-width: 70px; height: 70px; }
}

/* 弹窗与其它 */
.event-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.85); backdrop-filter: blur(10px);
  display: flex; align-items: center; justify-content: center; z-index: 1000;
}
.event-card {
  background: #1a1a1a; border: 1px solid #ffd70033; padding: 40px; border-radius: 32px;
  max-width: 400px; text-align: center;
}
.event-title { color: #ffd700; margin-bottom: 20px; letter-spacing: 2px; font-size: 1.2rem; }
.event-story { color: #ccc; line-height: 1.8; margin-bottom: 30px; font-style: italic; }
.event-close { background: #ffd700; border: none; padding: 12px 40px; border-radius: 20px; font-weight: bold; cursor: pointer; }

.rename-modal-overlay { 
  position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 2000;
  display: flex; align-items: center; justify-content: center;
}
.rename-card { width: 320px; text-align: center; padding: 30px; }
.rename-card input { 
  width: 100%; background: #000; border: 1px solid #333; padding: 12px; 
  border-radius: 12px; color: #fff; margin: 20px 0; text-align: center;
}
.modal-btns { display: flex; gap: 10px; }
.modal-btns button { flex: 1; padding: 10px; border-radius: 8px; cursor: pointer; }
.btn-confirm { background: #c0392b; border: none; color: #fff; }
.btn-confirm.btn-danger { background: #c0392b; }
.btn-confirm:disabled { opacity: 0.5; cursor: not-allowed; }

.release-warning h3 { color: #c0392b; margin-bottom: 10px; }
.release-warning p { font-size: 0.85rem; color: #888; line-height: 1.6; margin-bottom: 20px; }
.release-pw-input { margin-bottom: 25px !important; }

/* 无龙状态样式 */
.empty-state { align-items: center; justify-content: center; }
.empty-card { 
  max-width: 500px; width: 100%; text-align: center; padding: 60px 40px; 
  display: flex; flex-direction: column; align-items: center; gap: 20px;
}
.empty-icon { font-size: 5rem; filter: drop-shadow(0 0 20px rgba(192,57,43,0.3)); animation: float 3s ease-in-out infinite; }
@keyframes float { 0%,100% { transform: translateY(0); } 50% { transform: translateY(-20px); } }

.empty-title { font-family: 'Noto Serif SC', serif; font-size: 2rem; color: #fff; margin: 0; }
.empty-desc { color: #888; line-height: 1.6; font-size: 0.95rem; }

.guide-steps { width: 100%; display: flex; flex-direction: column; gap: 15px; margin: 20px 0; }
.step { display: flex; align-items: center; gap: 15px; background: rgba(255,255,255,0.03); padding: 12px 20px; border-radius: 12px; text-align: left; }
.step-num { width: 24px; height: 24px; background: #c0392b; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 0.75rem; font-weight: bold; color: #fff; }
.step-txt { font-size: 0.85rem; color: #ccc; }

.btn-goto-plaza { 
  background: #c0392b; color: #fff; border: none; padding: 15px 40px; 
  border-radius: 30px; font-weight: bold; cursor: pointer; transition: 0.3s;
  font-size: 1rem; margin-top: 10px;
}
.btn-goto-plaza:hover { transform: scale(1.05); background: #e74c3c; }

@keyframes pop-in { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
</style>
