<template>
  <transition name="t-modal">
    <div class="modal-mask" @click.self="$emit('close')">
      <div class="modal-card card-fortune">
        <div class="f-header">
          <div class="f-date">◈ {{ fortune.date }} ◈</div>
          <div class="f-luck-badge" :class="luckClass">{{ luckText }}</div>
        </div>
        
        <div class="f-verse-box">
          <div class="f-verse">{{ fortune.verse }}</div>
          <div class="f-interpretation">{{ fortune.interpretation }}</div>
        </div>
        
        <div class="f-grid">
          <div class="f-column suit">
            <div class="f-label">宜</div>
            <div class="f-items">
              <div v-for="item in suitItems" :key="item" class="f-item">{{ item }}</div>
            </div>
          </div>
          <div class="f-column avoid">
            <div class="f-label">不宜</div>
            <div class="f-items">
              <div v-for="item in avoidItems" :key="item" class="f-item">{{ item }}</div>
            </div>
          </div>
        </div>
        
        <div class="f-footer">
          <div class="f-task-tip">※ 已完成今日求签修行任务</div>
          <button class="btn-p f-btn" @click="$emit('close')">谨记真言</button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  fortune: Object
});

const suitItems = computed(() => props.fortune.suit ? props.fortune.suit.split(',') : []);
const avoidItems = computed(() => props.fortune.avoid ? props.fortune.avoid.split(',') : []);

// 过滤掉图标，只保留文字用于类名判断
const luckText = computed(() => {
    if (!props.fortune.luck) return '';
    return props.fortune.luck.replace(/[^\u4e00-\u9fa5]/g, '').trim();
});

const luckClass = computed(() => {
  const t = luckText.value;
  if (t.includes('大吉')) return 'l-great';
  if (t.includes('小吉') || t.includes('中吉')) return 'l-good';
  if (t.includes('平')) return 'l-normal';
  if (t.includes('凶')) return 'l-bad';
  return '';
});
</script>

<style scoped>
.modal-mask { position: fixed; inset: 0; background: rgba(0,0,0,0.9); backdrop-filter: blur(15px); z-index: 2000; display: flex; align-items: center; justify-content: center; }
.modal-card { width: 95%; max-width: 480px; background: #080808; border: 1px solid rgba(192,57,43,0.2); border-radius: 40px; padding: 40px; box-shadow: 0 40px 100px rgba(0,0,0,1); position: relative; overflow: hidden; }

.modal-card::before { content: ''; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%; background: radial-gradient(circle, rgba(192,57,43,0.05) 0%, transparent 70%); pointer-events: none; }

.f-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 35px; position: relative; z-index: 2; }
.f-date { font-size: 0.85rem; color: #444; letter-spacing: 2px; }
.f-luck-badge { padding: 6px 20px; border-radius: 12px; font-weight: 900; font-size: 1.2rem; letter-spacing: 3px; }

.l-great { background: #c0392b; color: #fff; box-shadow: 0 0 30px rgba(192,57,43,0.5); }
.l-good { background: #d35400; color: #fff; }
.l-normal { background: #333; color: #888; }
.l-bad { background: #1a1a1a; color: #555; border: 1px solid #333; }

.f-verse-box { text-align: center; margin-bottom: 40px; padding: 30px 20px; background: rgba(255,255,255,0.02); border-radius: 24px; border: 1px solid rgba(255,255,255,0.03); position: relative; z-index: 2; }
.f-verse { font-family: 'Noto Serif SC', serif; font-size: 2.2rem; color: #fff; margin-bottom: 20px; letter-spacing: 8px; font-weight: 900; }
.f-interpretation { font-size: 0.95rem; color: #aaa; line-height: 1.8; font-style: italic; max-width: 300px; margin: 0 auto; }

.f-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 40px; position: relative; z-index: 2; }
.f-column { padding: 25px 20px; border-radius: 24px; position: relative; }
.f-column.suit { background: rgba(39, 174, 96, 0.03); border: 1px solid rgba(39, 174, 96, 0.1); }
.f-column.avoid { background: rgba(192, 57, 43, 0.03); border: 1px solid rgba(192, 57, 43, 0.1); }

.f-label { font-weight: 900; font-size: 1.4rem; margin-bottom: 20px; text-align: center; }
.suit .f-label { color: #27ae60; text-shadow: 0 0 10px rgba(39, 174, 96, 0.3); }
.avoid .f-label { color: #c0392b; text-shadow: 0 0 10px rgba(192, 57, 43, 0.3); }

.f-items { display: flex; flex-direction: column; gap: 12px; }
.f-item { font-size: 0.95rem; color: #eee; text-align: center; font-weight: 500; }

.f-footer { text-align: center; position: relative; z-index: 2; }
.f-task-tip { font-size: 0.75rem; color: #27ae60; margin-bottom: 20px; opacity: 0.8; letter-spacing: 1px; }

.btn-p { width: 100%; background: #c0392b; color: #fff; border: none; border-radius: 16px; padding: 18px; font-weight: bold; cursor: pointer; transition: .4s; font-size: 1.1rem; letter-spacing: 4px; }
.btn-p:hover { background: #e74c3c; transform: translateY(-3px); box-shadow: 0 10px 30px rgba(192,57,43,0.3); }

.t-modal-enter-active, .t-modal-leave-active { transition: all 0.6s cubic-bezier(0.16, 1, 0.3, 1); }
.t-modal-enter-from, .t-modal-leave-to { opacity: 0; transform: scale(0.85) translateY(40px); filter: blur(10px); }
</style>
