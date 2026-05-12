<template>
  <div class="view-home">
    <div class="home-content">
      <!-- 桌面端左侧 / 移动端上方：文字区 -->
      <section class="text-section">
        <div class="quote-container">
          <div class="thinking-label" v-if="isThinking">
            <span class="pulse-dot"></span>
            {{ thinkingText }}
          </div>
          
          <div class="quote-main" :class="quote.type">
            <span class="q-mark open">“</span>
            <h1 class="q-content">{{ displayedQuote }}</h1>
            <span class="q-mark close">”</span>
          </div>

          <transition name="fade">
            <p class="quote-explain" v-if="showExplain">{{ quote.explain }}</p>
          </transition>
        </div>

        <transition name="pop">
          <div class="action-section" v-if="showEnter">
            <button class="btn-enter" @click="$emit('enter')">
              <span class="btn-shimmer"></span>
              探寻遗境
            </button>
            <button class="btn-fortune" @click="$emit('get-fortune')" :disabled="isShaking">
              {{ isShaking ? '灵力感应中...' : '求取灵语' }}
            </button>
            <button class="btn-sub" @click="$emit('show-about')">关于龙屿</button>
          </div>
        </transition>
      </section>

      <!-- 桌面端右侧 / 移动端下方：视觉区 -->
      <section class="visual-section">
        <div class="dragon-eye-wrap">
          <div class="eye-glow"></div>
          <div class="eye-inner">
             <img src="https://xiaolongya.cn/uploads/1778566530694964727.jpg" class="eye-img">
          </div>
          <div class="eye-ring r1"></div>
          <div class="eye-ring r2"></div>
          <div class="eye-ring r3"></div>
        </div>
      </section>
    </div>

    <!-- 底部微光 -->
    <div class="bottom-aurora"></div>
  </div>
</template>

<script setup>
defineProps({
  isThinking: Boolean,
  thinkingText: String,
  quote: Object,
  displayedQuote: String,
  showExplain: Boolean,
  showEnter: Boolean,
  isShaking: Boolean
});
defineEmits(['enter', 'show-about', 'get-fortune']);
</script>

<style scoped>
.view-home { 
  flex: 1; display: flex; align-items: center; justify-content: center; 
  padding: 60px; position: relative; overflow-y: auto; overflow-x: hidden;
}

.home-content { 
  display: flex; width: 100%; max-width: 1400px; gap: 80px; align-items: center; 
  z-index: 10;
}

/* 文字区 */
.text-section { flex: 1.2; }

.quote-container { margin-bottom: 60px; }

.thinking-label { 
  font-size: 0.8rem; color: #c0392b; letter-spacing: 4px; font-weight: bold; 
  margin-bottom: 30px; display: flex; align-items: center; gap: 10px;
  animation: fade-in 1s ease;
}
.pulse-dot { width: 8px; height: 8px; background: #c0392b; border-radius: 50%; animation: pulse-red 2s infinite; }
@keyframes pulse-red { 0% { box-shadow: 0 0 0 0 rgba(192,57,43,0.7); } 70% { box-shadow: 0 0 0 10px rgba(192,57,43,0); } 100% { box-shadow: 0 0 0 0 rgba(192,57,43,0); } }

.quote-main { position: relative; padding: 20px 60px; }
.q-content { 
  font-family: 'Noto Serif SC', serif; font-size: 3.5rem; color: #fff; line-height: 1.4; 
  letter-spacing: 4px; font-weight: 600;
  text-shadow: 0 10px 30px rgba(0,0,0,0.5);
  position: relative; z-index: 2;
}
.q-mark { 
  position: absolute; font-size: 10rem; font-family: "Georgia", serif; 
  color: #c0392b; opacity: 0.08; user-select: none; z-index: 1;
}
.q-mark.open { left: -10px; top: -50px; }
.q-mark.close { right: -10px; bottom: -60px; }

.quote-explain { 
  margin-top: 40px; font-size: 1.2rem; color: #888; font-style: italic; 
  line-height: 1.8; padding-left: 4px; border-left: 2px solid rgba(192,57,43,0.3);
}

.action-section { display: flex; gap: 24px; align-items: center; }

.btn-enter { 
  position: relative; background: #c0392b; color: #fff; border: none; 
  padding: 16px 48px; border-radius: 50px; font-size: 1.1rem; font-weight: bold; 
  cursor: pointer; overflow: hidden; transition: .4s;
  box-shadow: 0 15px 30px rgba(192,57,43,0.3);
}
.btn-enter:hover { transform: translateY(-5px); box-shadow: 0 20px 40px rgba(192,57,43,0.5); }
.btn-shimmer { 
  position: absolute; inset: 0; 
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  transform: translateX(-100%); animation: shimmer 3s infinite;
}
@keyframes shimmer { 100% { transform: translateX(100%); } }

.btn-fortune { 
  background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #c0392b; 
  padding: 14px 32px; border-radius: 50px; cursor: pointer; transition: .3s; font-weight: bold;
}
.btn-fortune:hover { background: #c0392b; color: #fff; box-shadow: 0 0 20px rgba(192,57,43,0.4); }

.btn-sub { background: none; border: 1px solid rgba(255,255,255,0.1); color: #555; padding: 14px 32px; border-radius: 50px; cursor: pointer; transition: .3s; }
.btn-sub:hover { border-color: #fff; color: #fff; }

/* 视觉区 */
.visual-section { flex: 0.8; display: flex; justify-content: center; }

.dragon-eye-wrap { position: relative; width: 320px; height: 320px; }
.eye-inner { 
  position: absolute; inset: 40px; border-radius: 50%; overflow: hidden; 
  border: 4px solid rgba(192,57,43,0.3); z-index: 5;
  animation: float 6s ease-in-out infinite;
}
.eye-img { width: 100%; height: 100%; object-fit: cover; filter: brightness(1.2); }
.eye-glow { 
  position: absolute; inset: 20px; background: radial-gradient(circle, rgba(192,57,43,0.4) 0%, transparent 70%); 
  filter: blur(20px); animation: pulse-glow 4s infinite;
}

.eye-ring { 
  position: absolute; border-radius: 50%; border: 1px solid rgba(192,57,43,0.1); 
  animation: spin 20s linear infinite;
}
.r1 { inset: 0; border-style: dashed; }
.r2 { inset: -20px; border-style: solid; opacity: 0.5; animation-direction: reverse; }
.r3 { inset: -40px; border-style: dotted; opacity: 0.3; }

@keyframes float { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-20px)} }
@keyframes pulse-glow { 0%,100%{opacity:0.4;transform:scale(1)} 50%{opacity:0.8;transform:scale(1.1)} }
@keyframes spin { to { transform: rotate(360deg); } }

.bottom-aurora { 
  position: absolute; bottom: -100px; left: -100px; right: -100px; height: 300px;
  background: radial-gradient(ellipse at bottom, rgba(192,57,43,0.15) 0%, transparent 70%);
  filter: blur(50px); pointer-events: none;
}

/* --- 响应式适配 --- */
@media (max-width: 1100px) {
  .q-content { font-size: 2.8rem; }
  .dragon-eye-wrap { width: 260px; height: 260px; }
}

@media (max-width: 900px) {
  .view-home { padding: 40px 20px; align-items: flex-start; }
  .home-content { flex-direction: column; gap: 60px; text-align: center; }
  .text-section { order: 1; width: 100%; }
  .visual-section { order: 2; width: 100%; padding-bottom: 40px; }
  
  .quote-main { padding: 0; }
  .q-content { font-size: 2.2rem; }
  .q-mark { display: none; }
  .quote-explain { border-left: none; padding-left: 0; }
  .action-section { justify-content: center; }
  
  .dragon-eye-wrap { width: 200px; height: 200px; }
  .eye-inner { inset: 20px; }
}

.fade-enter-active, .fade-leave-active { transition: opacity 1s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.pop-enter-active { transition: all 0.8s cubic-bezier(0.16, 1, 0.3, 1); }
.pop-enter-from { opacity: 0; transform: translateY(20px); }

@keyframes fade-in { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }
</style>
