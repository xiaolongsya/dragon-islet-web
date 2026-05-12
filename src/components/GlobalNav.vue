<template>
  <div class="global-nav" :class="{'nav-mobile': isMobile}">
    <!-- 桌面端文字标识 -->
    <div v-if="!isMobile" class="nav-logo-text" @click="$emit('update:modelValue', 'home')">
      <div class="logo-txt-main">龙屿</div>
      <div class="logo-txt-sub">Islet</div>
    </div>

    <div class="nav-menu">
      <!-- 桌面端动态指示器 -->
      <div v-if="!isMobile" class="nav-indicator" :style="indicatorStyle"></div>
      
      <div class="nav-item" :class="{active: modelValue==='home'}" @click="$emit('update:modelValue', 'home')">
        <div class="icon-box">
          <img class="n-icon" src="https://xiaolongya.cn/uploads/1778566503762301870.jpg">
          <div class="icon-glow"></div>
        </div>
        <span class="n-txt">岛屿</span>
      </div>

      <div class="nav-item" :class="{active: modelValue==='chat'}" @click="$emit('update:modelValue', 'chat')">
        <div class="icon-box">
          <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565720509204937.jpg">
          <div class="icon-glow"></div>
        </div>
        <span class="n-txt">誓约</span>
      </div>

      <div class="nav-item" :class="{active: modelValue==='raising'}" @click="isLoggedIn ? $emit('update:modelValue', 'raising') : $emit('open-modal', 'login')">
        <div class="icon-box">
          <img class="n-icon" src="https://xiaolongya.cn/uploads/1778608070094202474.jpg">
          <div class="icon-glow"></div>
        </div>
        <span class="n-txt">养成</span>
      </div>

      <div class="nav-item" :class="{active: modelValue==='archives'}" @click="$emit('update:modelValue', 'archives')">
        <div class="icon-box">
          <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565737832547358.jpg">
          <div class="icon-glow"></div>
        </div>
        <span class="n-txt">史诗</span>
      </div>

      <div class="nav-item" :class="{active: modelValue==='feedback'}" @click="isLoggedIn ? $emit('update:modelValue', 'feedback') : $emit('open-modal', 'login')">
        <div class="icon-box">
          <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565758375260099.jpg">
          <div class="icon-glow"></div>
        </div>
        <span class="n-txt">鳞笺</span>
      </div>

      <!-- 管理员守护菜单 -->
      <div v-if="isLoggedIn && user.role==='admin'" class="nav-item" :class="{active: modelValue==='admin'}" @click="$emit('update:modelValue', 'admin')">
        <div class="icon-box">
          <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565766078871519.jpg">
          <div class="icon-glow"></div>
        </div>
        <span class="n-txt">守护</span>
      </div>

      <!-- 手机端头像/登录按钮 -->
      <div v-if="isMobile" class="nav-item nav-user-mobile">
        <div v-if="!isLoggedIn" class="icon-box" @click="$emit('open-modal', 'login')">
           <div class="login-placeholder">登</div>
        </div>
        <div v-else class="icon-box" :class="{active: modelValue==='my-oaths'}" @click="$emit('update:modelValue', 'my-oaths')">
          <img :src="user.avatar || defAv" class="nav-av">
        </div>
        <span class="n-txt">{{ isLoggedIn ? '我的' : '登录' }}</span>
      </div>
    </div>

    <!-- 桌面端侧边底部用户信息 -->
    <div v-if="!isMobile" class="nav-user">
      <button v-if="!isLoggedIn" @click="$emit('open-modal', 'login')" class="btn-login-sm">登</button>
      <div v-else class="user-zone" :class="{active: modelValue==='my-oaths'}" @click="$emit('update:modelValue', 'my-oaths')">
        <div class="user-av-wrap">
          <img :src="user.avatar || defAv" class="nav-av">
        </div>
        <div class="user-mini-title">{{ user.title?.slice(0,2) || '游侠' }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, onMounted, watch } from 'vue';

const props = defineProps({
  modelValue: String,
  isLoggedIn: Boolean,
  user: Object,
  defAv: String,
  isMobile: Boolean
});
const emit = defineEmits(['update:modelValue', 'open-modal']);

const indicatorTop = ref(0);
const indicatorOpacity = ref(0);

const updateIndicator = () => {
  if (props.isMobile) return;
  // 增加延迟，确保 DOM 已经根据权限渲染完成
  setTimeout(() => {
    const activeEl = document.querySelector('.nav-item.active');
    if (activeEl) {
      indicatorTop.value = activeEl.offsetTop + (activeEl.offsetHeight / 2) - 10;
      indicatorOpacity.value = 1;
    } else {
      indicatorOpacity.value = 0;
    }
  }, 100);
};

const indicatorStyle = computed(() => ({
  transform: `translateY(${indicatorTop.value}px)`,
  opacity: indicatorOpacity.value
}));

watch(() => props.modelValue, () => {
  updateIndicator();
});

// 当用户信息（权限）变化时，也重新计算指示器位置
watch(() => props.user, () => {
  updateIndicator();
}, { deep: true });

onMounted(updateIndicator);
</script>

<style scoped>
.global-nav { 
  width: 72px; flex-shrink: 0; background: rgba(5,5,5,0.9); 
  border-right: 1px solid rgba(255,255,255,0.05); 
  display: flex; flex-direction: column; align-items: center; padding: 24px 0; 
  z-index: 1000; contain: paint;
  backdrop-filter: blur(20px);
}

.nav-logo-wrap { margin-bottom: 48px; cursor: pointer; transition: .4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.nav-logo-wrap:hover { transform: scale(1.1) rotate(5deg); }
.logo-stack { position: relative; width: 44px; height: 44px; border-radius: 12px; overflow: hidden; border: 1px solid rgba(192,57,43,0.3); background: #000; box-shadow: 0 0 20px rgba(192,57,43,0.1); }
.logo-base, .logo-wake { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; transition: opacity 0.5s ease; }
.logo-wake { opacity: 0; z-index: 2; filter: drop-shadow(0 0 8px #c0392b); }
.nav-logo-wrap:hover .logo-wake { opacity: 1; }
.logo-base { z-index: 1; animation: pulse-logo 4s ease-in-out infinite; }
@keyframes pulse-logo { 0%,100%{filter:brightness(0.8);} 50%{filter:brightness(1.2) contrast(1.1);} }

.nav-menu { flex: 1; display: flex; flex-direction: column; gap: 28px; width: 100%; position: relative; }

.nav-indicator {
  position: absolute; left: 0; width: 3px; height: 20px;
  background: #c0392b; box-shadow: 0 0 10px #c0392b;
  border-radius: 0 4px 4px 0; transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1);
  z-index: 5;
}

.nav-item { 
  display: flex; flex-direction: column; align-items: center; gap: 6px; 
  color: #555; cursor: pointer; transition: 0.3s; 
  position: relative; z-index: 10;
}
.icon-box { position: relative; width: 36px; height: 36px; display: flex; align-items: center; justify-content: center; }
.n-icon, .nav-av, .login-placeholder { 
  width: 32px; height: 32px; border-radius: 10px; object-fit: cover; 
  border: 1px solid rgba(255,255,255,0.05); transition: 0.4s cubic-bezier(0.16, 1, 0.3, 1);
  background: rgba(255,255,255,0.02);
}
.login-placeholder { display: flex; align-items: center; justify-content: center; color: #c0392b; font-weight: bold; border-color: rgba(192,57,43,0.3); }

.icon-glow {
  position: absolute; inset: -4px; background: radial-gradient(circle, rgba(192,57,43,0.4) 0%, transparent 70%);
  opacity: 0; transition: 0.4s; pointer-events: none;
}

.nav-item:hover { color: #888; }
.nav-item.active { color: #c0392b; }
.nav-item.active .n-icon, .nav-item.active .nav-av { border-color: #c0392b; box-shadow: 0 0 15px rgba(192,57,43,0.4); transform: scale(1.1); }
.nav-item.active .icon-glow { opacity: 1; }

.n-txt { font-size: 0.65rem; font-weight: 600; letter-spacing: 1px; opacity: 0.6; }
.nav-item.active .n-txt { opacity: 1; }

.nav-user { margin-top: auto; }
.btn-login-sm { width: 44px; height: 44px; border-radius: 14px; background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #c0392b; cursor: pointer; font-weight: bold; transition: .3s; }
.btn-login-sm:hover { background: #c0392b; color: #fff; transform: translateY(-3px); }

.user-zone { display: flex; flex-direction: column; align-items: center; gap: 8px; cursor: pointer; transition: .3s; }
.user-av-wrap { width: 44px; height: 44px; border-radius: 14px; overflow: hidden; border: 2px solid transparent; transition: .3s; }
.user-zone.active .user-av-wrap { border-color: #c0392b; box-shadow: 0 0 15px rgba(192,57,43,0.4); }
.user-mini-title { font-size: 0.6rem; color: #444; font-weight: bold; }

/* --- 手机端 Dock 栏 --- */
.nav-mobile {
  width: 100%; height: calc(64px + env(safe-area-inset-bottom));
  flex-direction: row; padding: 0 15px env(safe-area-inset-bottom);
  border-right: none; border-top: 1px solid rgba(255,255,255,0.05);
  position: fixed; bottom: 0; left: 0;
  box-shadow: 0 -10px 40px rgba(0,0,0,0.8);
}
.nav-mobile .nav-menu { flex-direction: row; justify-content: space-around; align-items: center; gap: 0; }
.nav-mobile .nav-item { gap: 4px; }
.nav-mobile .n-txt { font-size: 0.6rem; }
.nav-mobile .icon-box { width: 40px; height: 40px; }
.nav-mobile .n-icon, .nav-mobile .nav-av, .nav-mobile .login-placeholder { width: 28px; height: 28px; border-radius: 8px; }
.nav-mobile .nav-item.active { transform: translateY(-5px); }
</style>
