<template>
  <div class="view-raising">
    <div class="raising-container pop" v-if="loading">
      <div class="loading-state">
        <div class="loading-orb"></div>
        <span>探寻灵力波长...</span>
      </div>
    </div>

    <div class="raising-container" v-else-if="hasDragon">
      <!-- 手机端布局容器 (通过 CSS 控制在小屏幕显示) -->
      <div class="mobile-layout-branch">
        <!-- 顶部：状态条 (超紧凑) -->
        <div class="mobile-top-stats glass-card">
          <div class="m-stat" v-for="s in statsList" :key="s.label">
            <div class="m-stat-header">
              <span class="m-stat-label">{{ s.label }}</span>
              <button v-if="s.label === '成长值' && dragon.stage < 4 && dragon.exp >= nextExp" 
                      class="btn-m-evolve pulse-gold" @click="evolve">突破</button>
            </div>
            <div class="m-stat-bar-bg"><div class="m-stat-fill" :class="s.class" :style="{width: s.val + '%'}"></div></div>
          </div>
        </div>

        <!-- 手机端状态栏 (纯文字说明) -->
        <div class="status-bar-header-m" v-if="dragon.statuses && dragon.statuses.length">
          <span class="s-b-hint-m">✨ 特殊状态：影响奇遇概率与效果</span>
        </div>
        <div class="mobile-status-bar" v-if="dragon.statuses && dragon.statuses.length">
          <div class="m-status-tag" v-for="st in dragon.statuses" :key="st.id" @click="viewStatus(st)">
            <span class="m-st-dot"></span>
            <span class="m-st-name">{{ st.name }}</span>
            <span class="m-st-time">{{ formatTimeLeft(st.expires_at) }}</span>
          </div>
        </div>

        <!-- 中央：灵宠主视觉 (全屏感) -->
        <div class="mobile-hero-section">
          <div class="hero-name-row">
            <h2 class="hero-name">{{ dragon.name }} <span class="hero-rarity" :class="dragon.rarity">{{ rarityText }}</span></h2>
            <div class="hero-stage">{{ stageText }}</div>
          </div>

          <div class="hero-visual-container">
            <div class="hero-glow"></div>
            <div v-if="isGenerating" class="magic-loading-wrap mobile-magic-fix">
              <div class="magic-loading">
                <div class="orb-wrap"><div class="orb"></div></div>
                <div class="loading-text">灵力凝聚...</div>
              </div>
            </div>
            <div v-else-if="dragon.image_url && dragon.image_url !== '[GENERATING]'" class="hero-img-wrap">
              <img :src="dragon.image_url" class="hero-img">
            </div>
            <div v-else-if="!isGenerating" class="hero-magic-placeholder" @click="generateImage">
              <span>唤起真身</span>
            </div>

            <!-- 手机端气泡 -->
            <transition name="pop">
              <div v-if="displayText" class="dragon-speech-bubble mobile" @click="playDragonAudio">
                <div class="bubble-content">
                  <div class="bubble-main">
                    <span class="speaker-icon" :class="{ 'is-playing': isPlayingAudio }">🔊</span>
                    <span class="bubble-text">{{ displayText }}</span>
                  </div>
                  <span class="click-hint" v-if="!isPlayingAudio">(点击聆听龙语)</span>
                </div>
                <div class="bubble-arrow"></div>
              </div>
            </transition>
          </div>

          <!-- 核心操作 (移动到图片下方) -->
          <div class="hero-quick-actions">
            <div class="action-tile" @click="feed">投喂</div>
            <div class="action-tile" @click="play">陪玩</div>
            <div class="action-tile" @click="askGuide">求教</div>
            <div class="action-tile" @click="shareImage">展示</div>
          </div>
        </div>

        <!-- 底部：模块化导航 (触发抽屉) -->
        <div class="mobile-modular-nav">
          <div class="nav-btn" @click="showTasksDrawer = true">
            <span class="n-label">修行</span>
          </div>
          <div class="nav-btn" @click="showChatDrawer = true">
            <span class="n-label">私语</span>
          </div>
          <div class="nav-btn" @click="showInventoryDrawer = true">
            <span class="n-label">行囊</span>
          </div>
        </div>

        <!-- 所有抽屉面板 (Tasks, Chat, Inventory) -->
        <div class="drawers-gate">
          <!-- 任务抽屉 -->
          <transition name="drawer-slide">
            <div v-if="showTasksDrawer" class="drawer-mask" @click.self="showTasksDrawer = false">
              <div class="drawer-card tasks-drawer">
                <div class="drawer-h"><h3>每日修行</h3><button @click="showTasksDrawer = false">×</button></div>
                <div class="tasks-scroll">
                  <div v-for="t in tasks" :key="t.id" class="t-row">
                    <div class="t-title">{{ taskLabels[t.task_type] || '任务' }} ({{ t.progress }}/{{ t.max_progress }})</div>
                    <button v-if="t.progress >= t.max_progress && !t.is_claimed" class="btn-t-claim" @click="claimReward(t.id)">领赏</button>
                    <span v-else-if="t.is_claimed" class="t-done">已圆满</span>
                  </div>
                </div>
              </div>
            </div>
          </transition>

          <!-- 私语抽屉 -->
          <transition name="drawer-slide">
            <div v-if="showChatDrawer" class="drawer-mask" @click.self="showChatDrawer = false">
              <div class="drawer-card chat-drawer">
                <div class="drawer-h"><h3>灵魂私语</h3><button @click="showChatDrawer = false">×</button></div>
                <div class="chat-scroll-mobile" ref="mobileChatScroll">
                  <div v-for="(m, i) in dragonChat" :key="i" class="m-bubble" :class="m.role">{{ m.content }}</div>
                </div>
                <div class="chat-input-mobile">
                  <input v-model="chatInput" @keyup.enter="sendChat" placeholder="倾听它的心声...">
                  <button @click="sendChat">唤起</button>
                </div>
              </div>
            </div>
          </transition>

          <!-- 行囊抽屉 -->
          <transition name="drawer-slide">
            <div v-if="showInventoryDrawer" class="drawer-mask" @click.self="showInventoryDrawer = false">
              <div class="drawer-card inv-drawer">
                <div class="drawer-h"><h3>龙之行囊</h3><button @click="showInventoryDrawer = false">×</button></div>
                <div class="inv-grid-mobile">
                  <div v-for="item in items" :key="item.type" class="inv-slot" @click="selectedItem = item.type === selectedItem ? null : item.type">
                    <div class="inv-icon">{{ getItemIcon(item.type) }}</div>
                    <div class="inv-num">x{{ item.count }}</div>
                    <div class="inv-name">{{ getItemName(item.type) }}</div>
                    <div v-if="selectedItem === item.type && isUsable(item.type)" class="inv-use-btn" @click.stop="useItem(item.type)">使用</div>
                  </div>
                </div>
              </div>
            </div>
          </transition>
        </div>
      </div>

      <!-- 桌面端布局容器 (通过 CSS 控制在大屏幕显示) -->
      <div class="desktop-layout-branch">
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

        <div class="raising-grid">
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
            <div class="status-bar-header" v-if="dragon.statuses && dragon.statuses.length">
              <span class="s-b-title">活跃状态</span>
              <span class="s-b-hint">影响奇遇概率与效果</span>
            </div>
            <div class="global-status-tags" v-if="dragon.statuses && dragon.statuses.length">
              <div v-for="st in dragon.statuses" :key="st.id" class="status-tag-pill" @click="viewStatus(st)">
                <span class="pulse-dot"></span>
                <span class="st-main-info">{{ st.name }} ({{ formatTimeLeft(st.expires_at) }})</span>
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
                </div>
                <div v-else-if="!isGenerating" class="magic-entry" @click="generateImage">
                  <div class="magic-btn-inner">
                    <span class="m-text">唤起真身</span>
                    <span class="m-sub">MANIFEST SPIRIT</span>
                  </div>
                </div>

                <!-- 桌面端气泡 -->
                <transition name="pop">
                  <div v-if="displayText" class="dragon-speech-bubble desktop" @click="playDragonAudio">
                    <div class="bubble-content">
                      <div class="bubble-main">
                        <span class="speaker-icon" :class="{ 'is-playing': isPlayingAudio }">🔊</span>
                        <span class="bubble-text">{{ displayText }}</span>
                      </div>
                      <span class="click-hint" v-if="!isPlayingAudio">(点击聆听龙语)</span>
                    </div>
                    <div class="bubble-arrow"></div>
                  </div>
                </transition>
              </div>
            </div>

            <div class="quick-actions">
              <button class="act-btn" @click="feed" :disabled="isActing">投喂</button>
              <button class="act-btn" @click="play" :disabled="isActing">互动</button>
              <button class="act-btn" @click="askGuide" :disabled="isActing">求教</button>
              <button class="act-btn" @click="shareImage" :disabled="isActing">展示</button>
            </div>
          </div>

          <!-- 右侧：任务与对话 -->
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
                    <div class="t-name">{{ taskLabels[task.task_type] || '修行任务' }}</div>
                    <div class="t-progress-text">{{ task.progress }}/{{ task.max_progress }}</div>
                  </div>
                  <div class="t-action">
                    <button v-if="task.progress >= task.max_progress && !task.is_claimed" class="btn-claim-reward" @click="claimReward(task.id)">领赏</button>
                    <span v-else-if="task.is_claimed" class="t-status-done">已圆满</span>
                    <div v-else class="t-mini-bar"><div class="t-mini-fill" :style="{width: (task.progress/task.max_progress*100)+'%'}"></div></div>
                  </div>
                </div>
              </div>
            </div>

            <!-- 私语界面 -->
            <div v-else class="dragon-chat-wrap">
              <div class="soul-status">
                <div class="soul-personality">
                  <span class="s-p-label">灵魂特质:</span>
                  <span class="s-p-val">{{ dragon.soul || '纯净无瑕' }}</span>
                </div>
              </div>

              <div class="d-chat-messages" ref="chatScroll">
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

        <!-- 底部：行囊 (桌面端常驻) -->
        <div class="bottom-section glass-card pop">
          <div class="card-header">龙之囊中物</div>
          <div class="items-grid-scroll">
            <div v-if="items.length===0" class="item-empty">空空如也</div>
            <div v-for="item in items" :key="item.id" class="item-slot" :class="{ 'is-selected': selectedItem && selectedItem.id === item.id }" @click="selectedItem = (selectedItem && selectedItem.id === item.id) ? null : item">
              <div class="item-icon">{{ getItemIcon(item) }}</div>
              <div class="item-count">x{{ item.count }}</div>
              <div class="item-name-tag">{{ getItemName(item) }}</div>
              
              <transition name="fade">
                <div v-if="selectedItem && selectedItem.id === item.id" class="item-desc-bubble">
                  <div class="i-desc-text">{{ item.desc || '一件神秘的龙嗣珍宝。' }}</div>
                  <div v-if="isUsable(item)" class="use-overlay">
                    <button class="btn-use-inner" @click.stop="useItem(item)">使用</button>
                  </div>
                  <div v-else class="collect-tag">收藏品</div>
                </div>
              </transition>
            </div>
          </div>
        </div>
      </div>
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

    <!-- 状态详情弹窗 -->
    <transition name="pop">
      <div class="rename-modal-overlay" v-if="selectedStatus" @click.self="selectedStatus = null">
        <div class="status-detail-card glass-card pop">
          <div class="status-icon-large">✨</div>
          <h3>{{ selectedStatus.name }}</h3>
          <div class="status-desc">{{ selectedStatus.desc }}</div>
          <div class="status-effect-box">
            <div class="effect-label">灵力影响:</div>
            <div class="effect-val">{{ selectedStatus.effect || '暂无数据' }}</div>
          </div>
          <div class="status-duration">剩余时间: {{ formatTimeLeft(selectedStatus.expires_at) }}</div>
          <button class="btn-confirm" @click="selectedStatus = null">知晓了</button>
        </div>
      </div>
    </transition>

    <!-- 奇遇弹窗 (全局层级) -->
    <transition name="fade">
      <div v-if="randomEvent" class="event-overlay" @click="randomEvent = null">
        <div class="event-card" @click.stop>
          <div class="event-title">◈ 奇遇时刻 ◈</div>
          <div class="event-story">{{ randomEvent }}</div>
          <button class="event-close" @click="randomEvent = null">铭记于心</button>
        </div>
      </div>
    </transition>

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
const props = defineProps(['isLoggedIn', 'user', 'defAv', 'isMobile']);
const emit = defineEmits(['open-modal', 'switch-view']);

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
const mobileChatScroll = ref(null);

// 手机端抽屉状态
const showTasksDrawer = ref(false);
const showChatDrawer = ref(false);
const showInventoryDrawer = ref(false);

const getItemIcon = (item) => {
  if (item.name === '龙牙') return '🦴';
  if (item.name === '逆鳞') return '🔥';
  const icons = { 'food': '◆', 'exp_pill': '◈', 'sacrifice_stone': '◇', 'custom': '▣' };
  return icons[item.type] || '▣';
};

const getItemName = (item) => {
  if (item.type === 'custom' || item.name) return item.name;
  const names = { 'food': '龙粮', 'exp_pill': '龙髓丹', 'sacrifice_stone': '献祭之石' };
  return names[item.type] || '神秘物品';
};

const isUsable = (item) => {
  if (item.type === 'custom') return item.category === 'usable';
  return ['exp_pill', 'sacrifice_stone'].includes(item.type);
};

const useItem = async (item) => {
  if (isActing.value) return;
  isActing.value = true;
  try {
    const r = await axios.post('/raising/use-item', { type: item.type, name: item.name });
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
const selectedStatus = ref(null);
const speakText = ref('');
const displayText = ref('');
const audioUrl = ref('');
const isPlayingAudio = ref(false);
let currentAudio = null;
let typewriterTimer = null;

const startTypewriter = (text) => {
  if (typewriterTimer) clearInterval(typewriterTimer);
  displayText.value = '';
  let i = 0;
  typewriterTimer = setInterval(() => {
    if (i < text.length) {
      displayText.value += text[i];
      i++;
    } else {
      clearInterval(typewriterTimer);
    }
  }, 50); // 50ms 一个字，更灵动
};

const fetchDragonSpeak = async () => {
  try {
    const r = await axios.get('/raising/speak');
    speakText.value = r.data.text;
    audioUrl.value = r.data.audio_url;
    if (speakText.value) {
      startTypewriter(speakText.value);
    }
  } catch (e) {
    console.error('龙语感应失败:', e);
  }
};

const playDragonAudio = () => {
  console.log('尝试播放龙语:', audioUrl.value);
  if (!audioUrl.value) {
    alert('龙宝宝正处于深度冥想，暂时无法感应声音');
    return;
  }
  if (isPlayingAudio.value) return;
  
  if (currentAudio) currentAudio.pause();
  
  // 处理 localhost 映射问题
  let finalUrl = audioUrl.value;
  if (finalUrl.includes('localhost') && !window.location.hostname.includes('localhost')) {
    finalUrl = finalUrl.replace('localhost:8888', window.location.host);
  }

  currentAudio = new Audio(finalUrl);
  isPlayingAudio.value = true;
  
  currentAudio.play().catch(err => {
    console.error('音频播放失败:', err);
    isPlayingAudio.value = false;
    alert('听不到它的声音...请检查浏览器静音设置或尝试重新刷新。');
  });
  
  currentAudio.onended = () => {
    isPlayingAudio.value = false;
  };
};

const viewStatus = (st) => {
  selectedStatus.value = st;
};

const formatTimeLeft = (expiresAt) => {
  if (!expiresAt) return '永久';
  const now = new Date();
  const end = new Date(expiresAt);
  const diff = end - now;
  if (diff <= 0) return '即将消失';
  
  const hours = Math.floor(diff / (1000 * 60 * 60));
  const mins = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  if (hours > 0) return `${hours}时${mins}分`;
  return `${mins}分`;
};

const nextExp = computed(() => {
  if (dragon.value.stage >= 4) return 0; // 无上限
  const map = [100, 300, 800, 2000];
  return map[dragon.value.stage] || 5000;
});

const statsList = computed(() => [
  { label: '饱食度', icon: '🍖', val: (dragon.value.hunger/dragon.value.max_hunger)*100, text: `${dragon.value.hunger}/${dragon.value.max_hunger}`, class: 'hunger' },
  { label: '心情值', icon: '💖', val: (dragon.value.happiness/dragon.value.max_happiness)*100, text: `${dragon.value.happiness}/${dragon.value.max_happiness}`, class: 'happiness' },
  { label: '成长值', icon: '✨', val: dragon.value.stage < 4 ? (dragon.value.exp/nextExp.value)*100 : 100, text: dragon.value.stage < 4 ? `${dragon.value.exp}/${nextExp.value}` : 'MAX', class: 'exp' },
  { label: '今日机缘', icon: '🎐', val: (dragon.value.daily_event_count/10)*100, text: `${dragon.value.daily_event_count || 0}/10`, class: 'event' }
]);

const rarityText = computed(() => {
  const map = { 'common': '凡俗', 'rare': '珍稀', 'epic': '史诗' };
  return map[dragon.value.rarity] || '未知';
});

const stageText = computed(() => {
  const map = ['龙蛋', '幼龙', '青年龙', '壮年龙', '真龙'];
  return map[dragon.value.stage] || '幻化中';
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
    if (mobileChatScroll.value) {
      mobileChatScroll.value.scrollTop = mobileChatScroll.value.scrollHeight;
    }
  }, 100);
};

watch(activePanel, (newVal) => {
  if (newVal === 'chat') {
    scrollToBottom();
  }
});

watch(showChatDrawer, (newVal) => {
  if (newVal) {
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
  fetchDragonSpeak();
});
onBeforeUnmount(() => clearInterval(pollTimer));
</script>

<style scoped>
.view-raising { 
  flex: 1; display: flex; flex-direction: column; height: 100vh;
  position: relative; background: transparent;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

.raising-container { 
  max-width: 1200px; width: 100%; margin: 0 auto; 
  padding: 20px; display: flex; flex-direction: column; gap: 20px;
  background: radial-gradient(circle at top right, rgba(192,57,43,0.05), transparent 60%);
}

.loading-state {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  gap: 20px; color: #666; font-family: 'Noto Serif SC', serif; letter-spacing: 4px;
}
.loading-orb {
  width: 40px; height: 40px; border: 2px solid #c0392b; border-radius: 50%;
  border-top-color: transparent; animation: spin 1s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

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
.raising-grid { 
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
  border: 1px solid rgba(255,255,255,0.1); background: #000;
  position: relative; box-shadow: 0 40px 80px rgba(0,0,0,0.6);
  overflow: visible;
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
.image-overlay-actions {
  position: absolute; bottom: 20px; left: 0; right: 0;
  display: flex; justify-content: center; gap: 15px;
}
.btn-share-img-new, .btn-redraw-img {
  background: rgba(0,0,0,0.6); backdrop-filter: blur(10px);
  padding: 8px 20px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.2);
  color: #fff; font-size: 0.8rem; cursor: pointer; transition: 0.3s;
}
.btn-share-img-new:hover, .btn-redraw-img:hover { background: #c0392b; border-color: #c0392b; }

/* 手机端重塑按钮 */
.hero-img-wrap { width: 100%; height: 100%; position: relative; }
.btn-redraw-mobile {
  position: absolute; bottom: 15px; right: 15px;
  background: rgba(192,57,43,0.8); border: none; color: #fff;
  padding: 6px 14px; border-radius: 12px; font-size: 0.7rem; font-weight: bold;
}

/* 状态详情卡片 */
.status-detail-card { width: 300px; text-align: center; }
.status-icon-large { font-size: 3rem; margin-bottom: 15px; text-shadow: 0 0 20px #ffd70033; }
.status-desc { color: #888; font-size: 0.85rem; line-height: 1.6; margin: 15px 0; }
.status-effect-box { background: rgba(0,0,0,0.3); padding: 12px; border-radius: 12px; margin-bottom: 15px; }
.effect-label { font-size: 0.7rem; color: #555; margin-bottom: 4px; }
.effect-val { color: #2ecc71; font-weight: bold; font-family: 'Outfit', sans-serif; }
.status-duration { font-size: 0.75rem; color: rgba(255,255,255,0.3); margin-bottom: 20px; }

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

/* 桌面端布局适配 */
@media (max-width: 1000px) {
  .raising-grid { grid-template-columns: 1fr; }
  .tasks-section { height: 300px; }
  .top-status-bar { grid-template-columns: repeat(2, 1fr); gap: 15px; }
}

.drawer-slide-enter-active, .drawer-slide-leave-active { transition: transform 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.drawer-slide-enter-from, .drawer-slide-leave-to { transform: translateY(100%); }

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

/* 响应式分支控制 */
.mobile-layout-branch { display: none; }
.desktop-layout-branch { display: block; flex: 1; display: flex; flex-direction: column; gap: 20px; }

@media (max-width: 600px) {
  .desktop-layout-branch { display: none !important; }
  .mobile-layout-branch { 
    display: flex; flex-direction: column; height: 100vh; overflow: hidden; 
    padding: 0; gap: 0; background: #050505;
  }
  .raising-container.pop, .raising-container.empty-state {
    height: 100vh; display: flex; flex-direction: column; justify-content: center;
    background: #050505; padding: 20px;
  }
}

.mobile-top-stats {
  padding: 12px 15px; display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px 20px;
  background: rgba(15,15,15,0.8); border-radius: 0 0 20px 20px; border: none; border-bottom: 1px solid rgba(255,255,255,0.05);
}
.m-stat { display: flex; flex-direction: column; gap: 4px; }
.m-stat-header { display: flex; justify-content: space-between; align-items: center; }
.m-stat-label { font-size: 0.7rem; color: #888; white-space: nowrap; }
.btn-m-evolve { 
  background: linear-gradient(135deg, #ffd700, #ff8c00); color: #000; border: none;
  padding: 2px 8px; border-radius: 4px; font-weight: 900; font-size: 0.65rem;
  cursor: pointer;
}
.m-stat-bar-bg { width: 100%; height: 4px; background: rgba(255,255,255,0.05); border-radius: 10px; overflow: hidden; }
.m-stat-fill { height: 100%; border-radius: 10px; transition: 0.5s; }

.status-bar-header-m {
  padding: 8px 15px 0; background: rgba(0,0,0,0.2);
}
.s-b-hint-m { font-size: 0.65rem; color: #ff4d4d; }

.mobile-status-bar {
  display: flex; gap: 10px; overflow-x: auto; padding: 8px 15px 12px;
  background: rgba(0,0,0,0.2); border-bottom: 1px solid rgba(255,255,255,0.05);
}
.mobile-status-bar::-webkit-scrollbar { display: none; }
.m-status-tag {
  background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3);
  padding: 4px 10px; border-radius: 10px; display: flex; align-items: center; gap: 6px;
  flex-shrink: 0;
}
.m-st-dot { width: 4px; height: 4px; background: #ff4d4d; border-radius: 50%; box-shadow: 0 0 5px #ff4d4d; }
.m-st-name { font-size: 0.75rem; color: #ff4d4d; font-weight: bold; }
.m-st-time { font-size: 0.65rem; color: rgba(255,255,255,0.3); }

.mobile-hero-section {
  flex: 1; display: flex; flex-direction: column; position: relative; padding: 20px; min-height: 0;
}
.hero-name-row { text-align: center; margin-bottom: 15px; }
.hero-name { font-family: 'Noto Serif SC', serif; font-size: 1.6rem; color: #fff; margin: 0; }
.hero-rarity { font-size: 0.65rem; padding: 2px 8px; border-radius: 5px; vertical-align: middle; margin-left: 5px; }
.hero-stage { font-size: 0.8rem; color: #555; margin-top: 4px; }

.hero-visual-container {
  flex: 1; position: relative; display: flex; align-items: center; justify-content: center; min-height: 0;
  overflow: visible;
}
.hero-glow { position: absolute; width: 80%; height: 80%; background: radial-gradient(circle, rgba(192,57,43,0.15) 0%, transparent 70%); animation: hero-pulse 3s infinite; }
.hero-img { max-width: 100%; max-height: 100%; object-fit: contain; border-radius: 32px; filter: drop-shadow(0 20px 50px rgba(0,0,0,0.5)); }
.hero-magic-placeholder { width: 200px; height: 200px; border: 1px dashed #444; border-radius: 50%; display: flex; align-items: center; justify-content: center; color: #444; font-size: 0.9rem; }

.hero-quick-actions {
  display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; padding: 15px 0;
  margin-top: auto;
}
.action-tile {
  background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08);
  backdrop-filter: blur(10px); border-radius: 18px; height: 48px; display: flex; align-items: center; justify-content: center;
  color: #ccc; font-size: 0.85rem; font-weight: bold; cursor: pointer;
  box-shadow: 0 10px 20px rgba(0,0,0,0.2); transition: 0.3s;
}
.action-tile:active { transform: scale(0.9) rotate(5deg); background: rgba(192,57,43,0.2); color: #fff; border-color: #c0392b; }

.mobile-modular-nav {
  height: 80px; display: flex; gap: 1px; background: rgba(255,255,255,0.02);
  border-top: 1px solid rgba(255,255,255,0.05); padding-bottom: 10px;
}
.nav-btn { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 4px; }
.n-icon { font-size: 1.4rem; }
.n-label { font-size: 0.7rem; color: #666; font-weight: bold; }

/* 抽屉全局 */
.drawer-mask { position: fixed; inset: 0; background: rgba(0,0,0,0.85); backdrop-filter: blur(15px); z-index: 5000; }
.drawer-card { position: absolute; bottom: 0; left: 0; right: 0; background: #080808; border-top: 1px solid rgba(255,255,255,0.1); border-radius: 40px 40px 0 0; padding: 25px; box-shadow: 0 -20px 50px rgba(0,0,0,1); }
.drawer-h { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.drawer-h h3 { margin: 0; font-family: 'Noto Serif SC', serif; color: #fff; letter-spacing: 3px; }
.drawer-h button { background: none; border: none; color: #444; font-size: 2rem; cursor: pointer; }

.tasks-drawer { height: 65vh; }
.t-row { display: flex; justify-content: space-between; align-items: center; padding: 15px 0; border-bottom: 1px solid #111; }
.t-title { font-size: 0.9rem; color: #ccc; }
.btn-t-claim { background: #c0392b; color: #fff; border: none; padding: 6px 15px; border-radius: 8px; font-size: 0.8rem; }
.t-done { color: #27ae60; font-size: 0.8rem; }

.chat-drawer { height: 92vh; min-height: 500px; display: flex; flex-direction: column; border-top: 1px solid rgba(192,57,43,0.3); }
.chat-scroll-mobile { flex: 1; overflow-y: auto; display: flex; flex-direction: column; gap: 15px; padding: 10px 0; }
.m-bubble { max-width: 80%; padding: 12px 18px; border-radius: 20px; font-size: 0.95rem; line-height: 1.6; }
.m-bubble.dragon { align-self: flex-start; background: #111; color: #eee; border-radius: 5px 20px 20px 20px; border: 1px solid rgba(255,255,255,0.05); }
.m-bubble.user { align-self: flex-end; background: #c0392b; color: #fff; border-radius: 20px 5px 20px 20px; }
.chat-input-mobile { display: flex; gap: 10px; padding: 15px 20px 30px; background: #080808; }
.chat-input-mobile input { flex: 1; background: #1a1a1a; border: 1px solid #333; color: #fff; padding: 12px; border-radius: 12px; font-size: 1rem; }
.chat-input-mobile button { background: #c0392b; color: #fff; border: none; padding: 0 20px; border-radius: 12px; font-weight: bold; }

.inv-drawer { height: 70vh; }
.inv-grid-mobile { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; overflow-y: auto; }
.inv-slot { position: relative; background: #111; aspect-ratio: 1; border-radius: 20px; display: flex; flex-direction: column; align-items: center; justify-content: center; border: 1px solid #222; }
.inv-icon { font-size: 1.8rem; margin-bottom: 5px; }
.inv-num { font-size: 0.7rem; color: #c0392b; font-weight: 900; }
.inv-name { font-size: 0.65rem; color: #555; }
.inv-use-btn { position: absolute; inset: 0; background: rgba(192,57,43,0.9); display: flex; align-items: center; justify-content: center; color: #fff; font-weight: 900; border-radius: 20px; }

.mobile-magic-fix { position: absolute; inset: 0; background: transparent; z-index: 10; }
.mobile-magic-fix .orb-wrap { width: 80px; height: 80px; }
.mobile-magic-fix .loading-text { font-size: 0.8rem; }

@keyframes hero-pulse { 0%, 100% { transform: scale(1); opacity: 0.15; } 50% { transform: scale(1.2); opacity: 0.25; } }

@media (max-width: 600px) {
  .top-status-bar, .raising-grid, .bottom-section { display: none !important; }
}
.status-bar-header { 
  display: flex; justify-content: space-between; align-items: flex-end; 
  padding: 10px 20px 5px; border-bottom: 1px solid rgba(255,255,255,0.05);
}
.s-b-title { font-size: 0.85rem; color: #fff; font-weight: bold; }
.s-b-hint { font-size: 0.7rem; color: #c0392b; opacity: 0.8; }
.global-status-tags { display: flex; gap: 10px; padding: 15px 20px; flex-wrap: wrap; background: rgba(255,255,255,0.02); }

.status-bar-header-m { padding: 8px 15px 0; background: rgba(0,0,0,0.2); }
.s-b-hint-m { font-size: 0.65rem; color: #ff4d4d; }

/* 龙语气泡样式 */
.dragon-speech-bubble {
  position: absolute;
  z-index: 10000;
  cursor: pointer;
  filter: drop-shadow(0 10px 20px rgba(0,0,0,0.5));
  transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.dragon-speech-bubble:hover {
  transform: scale(1.05) translateY(-5px);
}

.dragon-speech-bubble.desktop {
  top: -70px;
  left: 50%;
  transform: translateX(-50%);
  width: auto;
  min-width: 200px;
  max-width: 380px;
}

.dragon-speech-bubble.mobile {
  top: -80px;
  left: 50%;
  transform: translateX(-50%);
  width: 95%;
}

.bubble-content {
  background: rgba(15, 15, 15, 0.8);
  backdrop-filter: blur(25px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  padding: 15px 20px;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #fff;
  text-align: center;
  box-shadow: 0 15px 35px rgba(0,0,0,0.5);
}

.bubble-main {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  width: 100%;
  justify-content: center;
}

.bubble-text {
  font-size: 0.95rem;
  line-height: 1.4;
  font-family: 'Noto Serif SC', serif;
  word-break: break-word;
  white-space: pre-wrap;
}

.speaker-icon {
  font-size: 1.2rem;
  flex-shrink: 0;
}

.speaker-icon.is-playing {
  animation: voice-bounce 0.5s infinite alternate;
}

.click-hint {
  font-size: 0.7rem;
  opacity: 0.5;
  white-space: nowrap;
}

.bubble-arrow {
  width: 0;
  height: 0;
  border-left: 10px solid transparent;
  border-right: 10px solid transparent;
  border-top: 10px solid rgba(255, 255, 255, 0.1);
  position: absolute;
  bottom: -9px;
  left: 50%;
  transform: translateX(-50%);
}

@keyframes voice-bounce {
  from { transform: scale(1); opacity: 0.7; }
  to { transform: scale(1.3); opacity: 1; }
}

/* 进场动画 */
.pop-enter-active {
  animation: pop-in 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.pop-leave-active {
  animation: pop-in 0.3s reverse ease-in;
}
@keyframes pop-in {
  0% { opacity: 0; transform: translateX(-50%) scale(0.5) translateY(20px); }
  100% { opacity: 1; transform: translateX(-50%) scale(1) translateY(0); }
}
</style>
