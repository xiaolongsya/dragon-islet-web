<template>
  <div class="global-nav">
    <div class="nav-logo-wrap nav-logo-interactive" @click="$emit('update:modelValue', 'about')">
      <div class="logo-stack">
        <img class="logo-base" src="https://xiaolongya.cn/uploads/1778566503762301870.jpg">
        <img class="logo-wake" src="https://xiaolongya.cn/uploads/1778566530694964727.jpg">
      </div>
    </div>
    <div class="nav-menu">
      <div class="nav-item" :class="{active: modelValue==='chat'}" @click="$emit('update:modelValue', 'chat')">
        <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565720509204937.jpg">
        <span class="n-txt">誓约</span>
      </div>
      <div class="nav-item" :class="{active: modelValue==='archives'}" @click="$emit('update:modelValue', 'archives')">
        <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565737832547358.jpg">
        <span class="n-txt">史诗</span>
      </div>
      <div class="nav-item" :class="{active: modelValue==='feedback'}" @click="isLoggedIn ? $emit('update:modelValue', 'feedback') : $emit('open-modal', 'login')">
        <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565758375260099.jpg">
        <span class="n-txt">鳞笺</span>
      </div>
      <div v-if="isLoggedIn && user.role==='admin'" class="nav-item" :class="{active: modelValue==='admin'}" @click="$emit('update:modelValue', 'admin')">
        <img class="n-icon" src="https://xiaolongya.cn/uploads/1778565766078871519.jpg">
        <span class="n-txt">守护</span>
      </div>
    </div>
    <div class="nav-user">
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
defineProps({
  modelValue: String,
  isLoggedIn: Boolean,
  user: Object,
  defAv: String
});
defineEmits(['update:modelValue', 'open-modal']);
</script>

<style scoped>
.global-nav { width: 72px; flex-shrink: 0; background: #050505; border-right: 1px solid rgba(255,255,255,0.05); display: flex; flex-direction: column; align-items: center; padding: 24px 0; z-index: 100; contain: paint; }
.nav-logo-wrap { margin-bottom: 48px; cursor: pointer; transition: .3s; }
.nav-logo-interactive:hover { transform: scale(1.1); }
.logo-stack { position: relative; width: 44px; height: 44px; border-radius: 12px; overflow: hidden; border: 1px solid rgba(192,57,43,0.2); background: #000; }
.logo-base, .logo-wake { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; transition: opacity 0.5s ease; }
.logo-wake { opacity: 0; z-index: 2; filter: drop-shadow(0 0 8px #c0392b); }
.nav-logo-wrap:hover .logo-wake { opacity: 1; }
.logo-base { z-index: 1; animation: pulse-logo 4s ease-in-out infinite; }
@keyframes pulse-logo { 0%,100%{filter:brightness(0.8) contrast(1);} 50%{filter:brightness(1.1) contrast(1.1);} }
.nav-menu { flex: 1; display: flex; flex-direction: column; gap: 24px; width: 100%; }
.nav-item { display: flex; flex-direction: column; align-items: center; gap: 8px; color: #555; cursor: pointer; transition: transform 0.3s, color 0.3s; will-change: transform; }
.nav-item:hover { color: #aaa; transform: translateY(-2px) translateZ(0); }
.nav-item.active { color: #c0392b; }
.n-icon { width: 32px; height: 32px; border-radius: 50%; object-fit: cover; border: 1px solid rgba(255,255,255,0.05); filter: grayscale(0.6) brightness(0.7); transition: filter 0.3s, box-shadow 0.3s; will-change: filter; }
.nav-item:hover .n-icon { filter: grayscale(0) brightness(1); box-shadow: 0 0 12px rgba(192,57,43,0.4); border-color: rgba(192,57,43,0.4); }
.nav-item.active .n-icon { filter: grayscale(0) brightness(1.2); box-shadow: 0 0 15px rgba(192,57,43,0.6); border-color: #c0392b; transform: scale(1.1); }
.n-txt { font-size: .6rem; font-weight: 600; letter-spacing: 2px; }
.nav-user { margin-top: auto; }
.btn-login-sm { width: 40px; height: 40px; border-radius: 12px; background: rgba(192,57,43,.1); border: 1px solid rgba(192,57,43,.3); color: #c0392b; cursor: pointer; font-weight: bold; transition: .2s; }
.btn-login-sm:hover { background: #c0392b; color: #fff; }
.user-zone { display: flex; flex-direction: column; align-items: center; gap: 6px; cursor: pointer; transition: .3s; opacity: 0.7; }
.user-zone:hover, .user-zone.active { opacity: 1; }
.user-zone.active .user-av-wrap { border-color: #c0392b; box-shadow: 0 0 12px rgba(192,57,43,0.4); }
.user-av-wrap { width: 44px; height: 44px; border-radius: 12px; overflow: hidden; border: 2px solid transparent; transition: .2s; }
.nav-av { width: 100%; height: 100%; object-fit: cover; }
.user-mini-title { font-size: 0.6rem; color: #444; letter-spacing: 1px; font-weight: 600; }
.user-zone.active .user-mini-title { color: #c0392b; }
</style>
