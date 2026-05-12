<template>
  <div class="view-raising">
    <div class="raising-container pop" v-if="loading">
      <div class="loading-state">
        <div class="loading-orb"></div>
        <span>探寻灵力波长...</span>
      </div>
    </div>

    <div class="raising-container" v-else>
      <!-- 无龙状态 -->
      <div class="empty-nest pop" v-if="!hasDragon">
        <div class="nest-visual">
          <div class="nest-glow"></div>
          <img src="https://xiaolongya.cn/uploads/1778566503762301870.jpg" class="nest-placeholder">
        </div>
        <h2 class="nest-title">龙巢虚位以待</h2>
        <p class="nest-hint">前往【誓约】向龙主祈求龙嗣，<br>开启属于你的守望之旅。</p>
      </div>


      <!-- 养成状态 -->
      <div class="dragon-layout" v-else>
        <!-- 左侧/上方：核心视觉区 -->
        <div class="visual-section pop">
          <div class="dragon-header">
            <div class="name-badge" @click="showRename=true">
              <span class="d-name">{{ dragon.name }}</span>
              <span class="d-rarity" :class="dragon.rarity">{{ rarityText }}</span>
            </div>
            <div class="d-stage">{{ stageText }}</div>
          </div>

          <div class="nest-core">
            <div class="dragon-visual-wrap" :class="[{ 'egg-stage': dragon.stage === 0 }, dragon.rarity]">
              <div class="visual-glow"></div>
              
              <!-- 正在幻化 -->
              <div v-if="isGenerating" class="magic-loading-wrap">
                <div class="magic-loading">
                  <div class="orb-wrap"><div class="orb"></div></div>
                  <div class="loading-text">灵力凝聚中...</div>
                </div>
              </div>

              <!-- 已有真身 -->
              <div v-else-if="dragon.image_url && dragon.image_url !== '[GENERATING]'" class="image-with-share">
                <img :src="dragon.image_url" class="dragon-img" @load="imgLoaded = true">
                <button class="btn-share-img" @click="shareImage" :disabled="isActing">
                  <span class="share-ic">✦</span> 分享真身
                </button>
              </div>
              
              <!-- 未幻化 -->
              <div v-else class="magic-entry" @click="generateImage">
                <div class="magic-pulse"></div>
                <div class="magic-btn-inner">
                  <span class="m-icon">🔮</span>
                  <span class="m-text">幻化真身</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 交互主按钮 -->
          <div class="quick-actions">
            <button class="act-btn" @click="feed" :disabled="isActing">
              <div class="btn-icon">🥩</div>
              <div class="btn-label">喂食</div>
            </button>
            <button class="act-btn" @click="play" :disabled="isActing">
              <div class="btn-icon">🧶</div>
              <div class="btn-label">陪玩</div>
            </button>
            <button class="act-btn" @click="askGuide" :disabled="isActing">
              <div class="btn-icon">📜</div>
              <div class="btn-label">求教</div>
            </button>
            <button class="act-btn" @click="share" :disabled="isActing">
              <div class="btn-icon">✨</div>
              <div class="btn-label">分享</div>
            </button>
          </div>
        </div>

        <!-- 右侧/下方：信息面板区 -->
        <div class="info-section">
          <!-- 状态条卡片 -->
          <div class="glass-card stats-card pop">
            <div class="card-title-mini">当前状态</div>
            <div class="stats-grid">
              <div class="stat-row">
                <div class="stat-info"><span>饱食度</span><span>{{ dragon.hunger }}%</span></div>
                <div class="progress-bg"><div class="progress-fill hunger" :style="{width: dragon.hunger+'%'}"></div></div>
              </div>
              <div class="stat-row">
                <div class="stat-info"><span>心情值</span><span>{{ dragon.happiness }}%</span></div>
                <div class="progress-bg"><div class="progress-fill happiness" :style="{width: dragon.happiness+'%'}"></div></div>
              </div>
              <div class="stat-row">
                <div class="stat-info"><span>成长值</span><span>{{ dragon.exp }}/{{ nextExp }}</span></div>
                <div class="progress-bg"><div class="progress-fill exp" :style="{width: (dragon.exp/nextExp)*100+'%'}"></div></div>
              </div>
            </div>
          </div>

          <!-- 每日修行卡片 -->
          <div class="glass-card tasks-card pop">
            <div class="card-header-flex">
              <span class="card-title-mini">每日修行</span>
              <span class="rarity-bonus" v-if="dragon.rarity !== 'common'">{{ dragon.rarity === 'epic' ? '5.0x' : '2.5x' }} 奖励</span>
            </div>
            <div class="tasks-list">
              <div v-if="tasks.length===0" class="task-empty">暂无修行任务，请联系龙主</div>
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

          <!-- 背包卡片 -->
          <div class="glass-card items-card pop">
            <div class="card-header">龙之囊中物</div>
            <div class="items-grid">
              <div v-if="items.length===0" class="item-empty">空空如也</div>
              <div v-for="item in items" :key="item.type" class="item-slot" @click="selectedItem = item.type === selectedItem ? null : item.type">
                <div class="item-icon">{{ getItemIcon(item.type) }}</div>
                <div class="item-count">x{{ item.count }}</div>
                <div class="item-name-tag">{{ getItemName(item.type) }}</div>
                
                <!-- 使用按钮 -->
                <transition name="pop">
                  <div v-if="selectedItem === item.type && isUsable(item.type)" class="use-popover">
                    <button class="btn-use" @click.stop="useItem(item.type)">使用</button>
                  </div>
                </transition>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 奇遇弹窗 -->
      <transition name="fade">
        <div v-if="randomEvent" class="event-overlay" @click="randomEvent = null">
          <div class="event-card" @click.stop>
            <div class="event-title">🏮 奇遇时刻 🏮</div>
            <div class="event-story">{{ randomEvent }}</div>
            <button class="event-close" @click="randomEvent = null">铭记于心</button>
          </div>
        </div>
      </transition>
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

const getItemIcon = (type) => {
  const icons = { 'food': '🥩', 'exp_pill': '💊', 'sacrifice_stone': '💎' };
  return icons[type] || '📦';
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
const newName = ref('');
const isGenerating = ref(false);
const imgLoaded = ref(false);

const rarityText = computed(() => {
  const map = { 'common': '凡俗', 'rare': '珍稀', 'epic': '史诗' };
  return map[dragon.value.rarity] || '未知';
});

const stageText = computed(() => {
  const map = ['静谧之蛋', '初生幼崽', '苍穹雏龙', '永恒巨龙'];
  return map[dragon.value.stage] || '幻化中';
});

const foodCount = computed(() => items.value.find(i => i.type === 'food')?.count || 0);
const nextExp = computed(() => (dragon.value.stage + 1) * 200);

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
  } catch(e) { console.error(e); }
  finally { loading.value = false; }
};

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
.view-raising { flex: 1; display: flex; flex-direction: column; overflow-y: auto; overflow-x: hidden; padding-bottom: 20px; }

.raising-container { max-width: 1200px; width: 100%; margin: 0 auto; padding: 40px 20px; }

/* 无龙状态 */
.empty-nest { text-align: center; padding: 80px 0; }
.nest-visual { position: relative; width: 240px; height: 240px; margin: 0 auto 40px; }
.nest-glow { position: absolute; inset: -20px; background: radial-gradient(circle, rgba(192,57,43,0.3) 0%, transparent 70%); animation: pulse 3s infinite; }
.nest-placeholder { width: 100%; height: 100%; border-radius: 40px; object-fit: cover; border: 2px solid rgba(255,255,255,0.05); }
.nest-title { font-family: 'Noto Serif SC', serif; font-size: 2rem; color: #fff; margin-bottom: 20px; letter-spacing: 4px; }
.nest-hint { color: #666; line-height: 1.8; font-size: 1.1rem; }

/* 养成主布局 */
.dragon-layout { display: grid; grid-template-columns: 1fr 400px; gap: 40px; align-items: flex-start; }

/* 视觉区 */
.visual-section { background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.05); border-radius: 32px; padding: 40px; backdrop-filter: blur(20px); position: sticky; top: 0; }

.dragon-header { display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 30px; }
.name-badge { cursor: pointer; display: flex; flex-direction: column; gap: 8px; }
.d-name { font-family: 'Noto Serif SC', serif; font-size: 2.2rem; color: #fff; text-shadow: 0 0 20px rgba(255,255,255,0.1); }
.d-rarity { font-size: 0.75rem; padding: 2px 10px; border-radius: 6px; width: fit-content; font-weight: 700; letter-spacing: 2px; }
.d-rarity.common { background: #555; color: #aaa; }
.d-rarity.rare { background: rgba(52,152,219,0.2); color: #3498db; border: 1px solid #3498db; }
.d-rarity.epic { background: rgba(192,57,43,0.2); color: #e74c3c; border: 1px solid #e74c3c; animation: rarity-glow 2s infinite; }
@keyframes rarity-glow { 0%,100% { box-shadow: 0 0 5px #c0392b; } 50% { box-shadow: 0 0 15px #c0392b; } }
.d-stage { color: #666; font-size: 0.9rem; font-weight: 600; letter-spacing: 2px; }

.nest-core { margin: 20px 0 40px; display: flex; justify-content: center; }
.dragon-visual-wrap { 
  position: relative; width: 100%; max-width: 500px; aspect-ratio: 1; 
  border-radius: 40px; overflow: hidden; border: 1px solid rgba(255,255,255,0.05); 
  background: #000; box-shadow: 0 30px 60px rgba(0,0,0,0.5);
  transform: translateZ(0);
}
.visual-glow { position: absolute; inset: 0; background: radial-gradient(circle at center, rgba(192,57,43,0.15) 0%, transparent 70%); z-index: 1; }

.dragon-img { width: 100%; height: 100%; object-fit: cover; animation: fade-in 1s ease; }
@keyframes fade-in { from { opacity: 0; transform: scale(1.05); } to { opacity: 1; transform: scale(1); } }

.btn-share-img { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.6); backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,0.2); color: #fff; padding: 10px 24px; border-radius: 30px; font-size: 0.85rem; cursor: pointer; transition: 0.3s; z-index: 10; }
.btn-share-img:hover { background: var(--primary); border-color: var(--primary); transform: translateX(-50%) translateY(-2px); }

.magic-entry { width: 100%; height: 100%; display: flex; align-items: center; justify-content: center; cursor: pointer; transition: 0.4s; }
.magic-entry:hover { background: rgba(192,57,43,0.05); }
.magic-btn-inner { text-align: center; z-index: 10; }
.m-icon { font-size: 4rem; display: block; margin-bottom: 10px; filter: drop-shadow(0 0 20px rgba(192,57,43,0.5)); }
.m-text { font-size: 1.2rem; font-weight: bold; color: #c0392b; letter-spacing: 4px; }

.quick-actions { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; }
.act-btn { background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.06); padding: 16px; border-radius: 20px; cursor: pointer; transition: 0.3s; color: #888; }
.act-btn:hover:not(:disabled) { background: rgba(192,57,43,0.1); border-color: rgba(192,57,43,0.3); color: #fff; transform: translateY(-4px); }
.btn-icon { font-size: 1.5rem; margin-bottom: 8px; }
.btn-label { font-size: 0.8rem; font-weight: 600; }
.btn-label small { color: #c0392b; margin-left: 4px; }

/* 信息区 */
.info-section { display: flex; flex-direction: column; gap: 24px; }
.glass-card { background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.05); border-radius: 24px; padding: 24px; backdrop-filter: blur(10px); }
.card-title-mini { font-size: 0.75rem; color: #555; font-weight: bold; letter-spacing: 2px; margin-bottom: 16px; text-transform: uppercase; }

.stats-grid { display: flex; flex-direction: column; gap: 16px; }
.stat-info { display: flex; justify-content: space-between; font-size: 0.85rem; margin-bottom: 6px; }
.progress-bg { height: 6px; background: rgba(255,255,255,0.03); border-radius: 3px; overflow: hidden; }
.progress-fill { height: 100%; transition: width 1s cubic-bezier(0.16, 1, 0.3, 1); }
.hunger { background: #e67e22; box-shadow: 0 0 10px rgba(230, 126, 34, 0.4); }
.happiness { background: #27ae60; box-shadow: 0 0 10px rgba(39, 174, 96, 0.4); }
.exp { background: #c0392b; box-shadow: 0 0 10px rgba(192, 57, 43, 0.4); }

.card-header-flex { display: flex; justify-content: space-between; margin-bottom: 16px; }
.rarity-bonus { font-size: 0.7rem; color: #c0392b; font-weight: bold; }
.tasks-list { display: flex; flex-direction: column; gap: 12px; min-height: 100px; }
.task-empty { text-align: center; color: #444; font-size: 0.8rem; padding: 20px 0; font-style: italic; }
.task-item { display: flex; justify-content: space-between; align-items: center; background: rgba(255,255,255,0.02); padding: 12px 16px; border-radius: 12px; border: 1px solid transparent; transition: .3s; }
.task-item.is-completed { border-color: rgba(39, 174, 96, 0.2); }
.task-item.is-claimed { opacity: 0.4; filter: grayscale(1); }
.t-name { font-size: 0.9rem; font-weight: 600; margin-bottom: 4px; }
.t-progress-text { font-size: 0.7rem; color: #444; }
.btn-claim-reward { background: #c0392b; border: none; color: #fff; font-size: 0.7rem; padding: 4px 12px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.t-status-done { font-size: 0.75rem; color: #27ae60; font-weight: bold; }
.t-mini-bar { width: 40px; height: 4px; background: #222; border-radius: 2px; overflow: hidden; }
.t-mini-fill { height: 100%; background: #555; }

.items-grid { display: flex; flex-wrap: wrap; gap: 12px; }
.item-slot { width: 60px; height: 60px; background: rgba(0,0,0,0.2); border: 1px solid rgba(255,255,255,0.05); border-radius: 12px; display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; }
.item-count { position: absolute; bottom: 4px; right: 4px; font-size: 0.6rem; color: #c0392b; font-weight: bold; }

/* 弹窗样式 */
.rename-modal-overlay { position: fixed; inset: 0; z-index: 10000; background: rgba(0,0,0,0.8); backdrop-filter: blur(20px); display: flex; align-items: center; justify-content: center; padding: 20px; }
.rename-card { width: 100%; max-width: 400px; padding: 40px; text-align: center; }
.rename-card h3 { font-family: 'Noto Serif SC', serif; color: #fff; margin-bottom: 24px; letter-spacing: 4px; }
.rename-card input { width: 100%; background: rgba(0,0,0,0.3); border: 1px solid rgba(255,255,255,0.1); padding: 16px; border-radius: 12px; color: #fff; font-size: 1.1rem; margin-bottom: 24px; text-align: center; outline: none; }
.rename-card input:focus { border-color: #c0392b; }
.modal-btns { display: flex; gap: 16px; }
.modal-btns button { flex: 1; padding: 12px; border-radius: 10px; font-weight: bold; cursor: pointer; transition: .3s; }
.btn-cancel { background: none; border: 1px solid #333; color: #666; }
.btn-confirm { background: #c0392b; border: none; color: #fff; }

/* --- 移动端适配 --- */
@media (max-width: 768px) {
  .raising-container { padding: 20px 15px; }
  .dragon-layout { grid-template-columns: 1fr; gap: 24px; }
  .visual-section { position: relative; padding: 24px; border-radius: 24px; }
  .d-name { font-size: 1.8rem; }
  .quick-actions { gap: 10px; }
  .act-btn { padding: 12px 8px; border-radius: 16px; }
  .btn-icon { font-size: 1.2rem; }
  .btn-label { font-size: 0.7rem; }
  .info-section { gap: 16px; }
  .glass-card { padding: 20px; border-radius: 20px; }
}

@keyframes pulse { 0%,100% { transform: scale(1); opacity: 0.3; } 50% { transform: scale(1.1); opacity: 0.5; } }
.pop { animation: pop-in 0.6s cubic-bezier(0.16, 1, 0.3, 1) both; }
@keyframes pop-in { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
/* 奇遇弹窗样式 */
.event-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(10px);
}

.event-card {
  background: linear-gradient(135deg, #2a2a2a 0%, #1a1a1a 100%);
  border: 1px solid rgba(255,215,0,0.3);
  border-radius: 24px;
  padding: 40px;
  max-width: 400px;
  text-align: center;
  box-shadow: 0 20px 50px rgba(0,0,0,0.5), 0 0 20px rgba(255,215,0,0.1);
}

.event-title {
  color: #ffd700;
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 20px;
  letter-spacing: 2px;
}

.event-story {
  color: #eee;
  font-size: 1.1rem;
  line-height: 1.8;
  margin-bottom: 30px;
  font-style: italic;
}

.event-close {
  background: #ffd700;
  color: #000;
  border: none;
  padding: 12px 40px;
  border-radius: 30px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}

.event-close:hover {
  transform: scale(1.05);
  box-shadow: 0 0 15px rgba(255,215,0,0.5);
}

/* 道具网格增强 */
.item-slot {
  position: relative;
  background: rgba(255,255,255,0.05);
  border-radius: 16px;
  padding: 15px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  transition: all 0.3s;
  cursor: pointer;
  border: 1px solid transparent;
}

.item-slot:hover {
  background: rgba(255,255,255,0.1);
  border-color: rgba(255,255,255,0.2);
}

.item-name-tag {
  font-size: 0.7rem;
  color: #888;
}

.use-popover {
  position: absolute;
  top: -45px;
  left: 50%;
  transform: translateX(-50%);
  background: #ffd700;
  padding: 5px;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.3);
  z-index: 10;
}

.btn-use {
  background: transparent;
  border: none;
  color: #000;
  font-size: 0.8rem;
  font-weight: bold;
  cursor: pointer;
  padding: 2px 10px;
}

/* 动画 */
.fade-enter-active, .fade-leave-active { transition: opacity 0.5s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.pop-enter-active { animation: pop-in 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.pop-leave-active { animation: pop-in 0.2s reverse; }
@keyframes pop-in {
  0% { transform: translateX(-50%) scale(0.5); opacity: 0; }
  100% { transform: translateX(-50%) scale(1); opacity: 1; }
}
</style>
