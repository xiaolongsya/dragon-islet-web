<template>
  <div class="view-home">
    <div class="landing-inner">
      <header class="landing-header">
        <div class="landing-logo-wrap" :class="{'logo-thinking': isThinking}">
          <div class="landing-logo-stack">
            <img class="l-base" src="https://xiaolongya.cn/uploads/1778566503762301870.jpg">
            <img class="l-wake" src="https://xiaolongya.cn/uploads/1778566530694964727.jpg">
          </div>
        </div>
        <div class="landing-title-group">
          <div class="landing-brand">DRAGON ISLET</div>
          <div class="landing-tagline">—— 龙屿 · 遗境誓约 ——</div>
        </div>
      </header>

      <div class="landing-accent-line"></div>

      <div class="thinking-wrap" v-if="isThinking">
        <div class="thinking-dots"><span></span><span></span><span></span></div>
        <div class="thinking-text">{{ thinkingText }}</div>
      </div>

      <div class="landing-quote-wrap" v-else :class="{'secret-glow': quote.type==='secret'}">
        <div class="landing-quote">
          <span class="q-mark">「</span>
          {{ displayedQuote }}<span class="cursor">_</span>
          <span class="q-mark">」</span>
        </div>
        <div class="landing-explain" v-if="showExplain">
          <span class="e-label">龙语释义：</span>{{ quote.explain }}
        </div>
      </div>

      <div class="landing-actions" v-if="showEnter">
        <button class="landing-enter" @click="$emit('enter')">
          踏入龙屿 <span class="arr">→</span>
        </button>
        <div class="landing-hint" v-if="quote.type==='secret'">✦ 隐秘彩蛋 ✦</div>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  isThinking: Boolean,
  thinkingText: String,
  quote: Object,
  displayedQuote: String,
  showExplain: Boolean,
  showEnter: Boolean
});
defineEmits(['enter']);
</script>

<style scoped>
.view-home { flex: 1; display: flex; align-items: center; justify-content: center; background: transparent; overflow-y: auto;}
.landing-inner{width:860px;padding:80px;display:flex;flex-direction:column;background: linear-gradient(135deg, rgba(15,15,15,0.85) 0%, rgba(0,0,0,0.4) 100%); backdrop-filter: blur(25px); border-radius: 40px; border: 1px solid rgba(255,255,255,0.08); box-shadow: 0 40px 120px rgba(0,0,0,0.9); }
.landing-header{display:flex;align-items:center;gap:48px;margin-bottom:48px;}
.landing-logo-wrap{width:140px;height:140px;flex-shrink:0;cursor:pointer;transition:0.6s;}
.landing-logo-stack { position: relative; width: 100%; height: 100%; border-radius: 36px; overflow: hidden; border: 2px solid rgba(192,57,43,0.3); background: #000; box-shadow: 0 0 40px rgba(0,0,0,0.8); }
.l-base, .l-wake { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; transition: opacity 0.8s cubic-bezier(0.4, 0, 0.2, 1); }
.l-wake { opacity: 0; z-index: 2; filter: drop-shadow(0 0 20px #c0392b) brightness(1.2); }
.landing-logo-wrap:hover .l-wake, .logo-thinking .l-wake { opacity: 1; }
.landing-logo-wrap:hover { transform: scale(1.05) rotate(2deg); }
.l-base { z-index: 1; animation: landing-pulse 5s ease-in-out infinite; }
@keyframes landing-pulse { 0%,100%{filter:brightness(0.7) contrast(1.1);transform:scale(1);} 50%{filter:brightness(1) contrast(1.2);transform:scale(1.02);} }
@keyframes pulse-fast{0%,100%{filter:drop-shadow(0 0 10px rgba(192,57,43,0.3));transform:scale(1)}50%{filter:drop-shadow(0 0 45px rgba(192,57,43,0.9));transform:scale(1.08)}}
.landing-accent-line{width:120px;height:4px;background:#c0392b;margin-bottom:60px; border-radius: 2px; box-shadow: 0 0 15px rgba(192,57,43,0.5);}
.thinking-wrap{display:flex;flex-direction:column;gap:32px;padding:40px 0;}
.thinking-dots{display:flex;gap:14px;}
.thinking-dots span{width:14px;height:14px;border-radius:50%;background:#c0392b;animation:dot-bounce 1.4s ease-in-out infinite; box-shadow: 0 0 10px rgba(192,57,43,0.4);}
.thinking-dots span:nth-child(2){animation-delay:.2s;}
.thinking-dots span:nth-child(3){animation-delay:.4s;}
@keyframes dot-bounce{0%,80%,100%{transform:scale(.6);opacity:.3}40%{transform:scale(1);opacity:1;}}
.thinking-text{font-size:1.2rem;color:#888;letter-spacing:4px; font-family: 'Noto Serif SC', serif;}
.landing-quote-wrap{width:100%;margin-bottom:60px; border-left: 3px solid rgba(192,57,43,0.3); padding-left: 40px;}
.secret-glow .landing-quote{color:#ffb8c6;text-shadow:0 0 24px rgba(255,150,180,.2);}
.landing-quote{font-family:'Noto Serif SC',serif;font-size:3rem;font-weight:600;color:#f0f0f0;line-height:1.75;letter-spacing:6px;text-align:left;}
.q-mark { color: #c0392b; font-weight: 700; margin: 0 10px; }
.cursor{animation:blink .8s step-end infinite;color:#c0392b;margin-left:4px;}
@keyframes blink{50%{opacity:0}}
.landing-explain{margin-top:28px;font-size:1.3rem;color:#888;line-height:2;letter-spacing:2px; animation:fade-up 0.8s ease both; font-style: italic;}
.e-label { color: #c0392b; font-weight: 600; margin-right: 12px; font-style: normal; font-size: 1rem; }
.landing-actions { display: flex; flex-direction: column; align-items: flex-start; margin-top: 40px; }
.landing-enter{background:transparent;border:1px solid rgba(192,57,43,0.4);color:#c0392b;padding:16px 56px;border-radius:12px;cursor:pointer;font-size:1.2rem;letter-spacing:6px;transition:all 0.4s;font-family:inherit; animation:fade-up 0.5s ease both; display: flex; align-items: center; gap: 20px;}
.landing-enter:hover{border-color:#c0392b;color:#eee;background:rgba(192,57,43,0.12); transform: translateX(10px); box-shadow: 0 0 40px rgba(192,57,43,0.3);}
.arr { transition: 0.3s; }
.landing-enter:hover .arr { transform: translateX(8px); }
.landing-hint{margin-top:24px;font-size:.9rem;color:rgba(255,150,180,.35);letter-spacing:8px;animation:fade-up .4s ease both;}
@keyframes fade-up{from{opacity:0;transform:translateY(15px)}to{opacity:1;transform:none}}
</style>
