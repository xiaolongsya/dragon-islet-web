<template>
  <div class="view-home">
    <div class="landing-inner">
      <header class="landing-header">
        <div class="landing-logo" :class="{'logo-thinking': isThinking}">🐉</div>
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
          {{ displayedQuote }}<span class="cursor">_</span>
        </div>
        <div class="landing-explain" v-if="showExplain">
          {{ quote.explain }}
        </div>
      </div>

      <button class="landing-enter" v-if="showEnter" @click="$emit('enter')">踏入龙屿 →</button>
      <div class="landing-hint" v-if="quote.type==='secret'&&showEnter">✦ 隐秘彩蛋 ✦</div>
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
.view-home { flex: 1; display: flex; align-items: center; justify-content: center; background: #050505; overflow-y: auto;}
.landing-inner{width:800px;padding:0;display:flex;flex-direction:column;}
.landing-header{display:flex;align-items:center;gap:24px;margin-bottom:32px;}
.landing-logo{font-size:3.5rem;color:#c0392b;filter:drop-shadow(0 0 16px rgba(192,57,43,.5));animation:pulse 3s ease-in-out infinite;flex-shrink:0;}
.logo-thinking{animation:pulse-fast 1s ease-in-out infinite !important;}
.landing-title-group{display:flex;flex-direction:column;gap:6px;}
.landing-brand{font-size:1.1rem;font-weight:700;letter-spacing:8px;color:#555;}
.landing-tagline{font-size:.9rem;color:#444;letter-spacing:3px;}
@keyframes pulse{0%,100%{filter:drop-shadow(0 0 16px rgba(192,57,43,.4))}50%{filter:drop-shadow(0 0 36px rgba(192,57,43,.8))}}
@keyframes pulse-fast{0%,100%{filter:drop-shadow(0 0 10px rgba(192,57,43,.3));transform:scale(1)}50%{filter:drop-shadow(0 0 45px rgba(192,57,43,.9));transform:scale(1.08)}}
.landing-accent-line{width:100%;height:1px;background:linear-gradient(90deg,#c0392b,rgba(192,57,43,0));margin-bottom:50px;}
.thinking-wrap{display:flex;flex-direction:column;gap:24px;padding:30px 0;}
.thinking-dots{display:flex;gap:14px;}
.thinking-dots span{width:12px;height:12px;border-radius:50%;background:#c0392b;animation:dot-bounce 1.4s ease-in-out infinite;}
.thinking-dots span:nth-child(2){animation-delay:.2s;}
.thinking-dots span:nth-child(3){animation-delay:.4s;}
@keyframes dot-bounce{0%,80%,100%{transform:scale(.6);opacity:.3}40%{transform:scale(1);opacity:1;}}
.thinking-text{font-size:1.1rem;color:#444;letter-spacing:3px;}
.landing-quote-wrap{width:100%;margin-bottom:48px;}
.secret-glow .landing-quote{color:#ffb8c6;text-shadow:0 0 24px rgba(255,150,180,.2);}
.landing-quote{font-family:'Noto Serif SC',serif;font-size:3.2rem;font-weight:600;color:#f0f0f0;line-height:1.75;letter-spacing:5px;text-align:left;}
.cursor{animation:blink .8s step-end infinite;color:#c0392b;margin-left:4px;}
@keyframes blink{50%{opacity:0}}
.landing-explain{margin-top:28px;font-size:1.2rem;color:#666;line-height:2;letter-spacing:2px;animation:fade-up .5s ease both;}
.landing-enter{align-self:flex-start;margin-top:50px;background:none;border:1px solid rgba(192,57,43,.35);color:rgba(192,57,43,.75);padding:16px 48px;border-radius:6px;cursor:pointer;font-size:1.1rem;letter-spacing:4px;transition:all .25s;font-family:inherit;animation:fade-up .4s ease both;}
.landing-enter:hover{border-color:#c0392b;color:#e0e0e0;background:rgba(192,57,43,.08);}
.landing-hint{margin-top:14px;font-size:.8rem;color:rgba(255,150,180,.35);letter-spacing:4px;animation:fade-up .4s ease both;}
@keyframes fade-up{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}
</style>
