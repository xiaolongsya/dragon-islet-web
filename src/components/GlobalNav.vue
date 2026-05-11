<template>
  <div class="global-nav">
    <div class="nav-logo-wrap">
      <span class="nav-logo-ic">🐉</span>
    </div>
    <div class="nav-menu">
      <div class="nav-item" :class="{active: modelValue==='chat'}" @click="$emit('update:modelValue', 'chat')">
        <span class="n-ic">✨</span><span class="n-txt">誓约</span>
      </div>
      <div class="nav-item" :class="{active: modelValue==='archives'}" @click="$emit('update:modelValue', 'archives')">
        <span class="n-ic">📜</span><span class="n-txt">史诗</span>
      </div>
      <div class="nav-item" :class="{active: modelValue==='my-oaths'}" @click="isLoggedIn ? $emit('update:modelValue', 'my-oaths') : $emit('open-modal', 'login')">
        <span class="n-ic">🔖</span><span class="n-txt">铭迹</span>
      </div>
      <div class="nav-item" :class="{active: modelValue==='feedback'}" @click="isLoggedIn ? $emit('update:modelValue', 'feedback') : $emit('open-modal', 'login')">
        <span class="n-ic">✉️</span><span class="n-txt">鳞笺</span>
      </div>
      <div v-if="isLoggedIn && user.role==='admin'" class="nav-item" :class="{active: modelValue==='admin'}" @click="$emit('update:modelValue', 'admin')">
        <span class="n-ic">🛡️</span><span class="n-txt">守护</span>
      </div>
    </div>
    <div class="nav-user">
      <button v-if="!isLoggedIn" @click="$emit('open-modal', 'login')" class="btn-login-sm">登</button>
      <div v-else class="user-av-wrap" @click="$emit('open-modal', 'profile')">
        <img :src="user.avatar || defAv" class="nav-av">
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
.global-nav { width: 72px; flex-shrink: 0; background: #050505; border-right: 1px solid rgba(255,255,255,0.05); display: flex; flex-direction: column; align-items: center; padding: 24px 0; z-index: 100; }
.nav-logo-wrap { margin-bottom: 48px; }
.nav-logo-ic { font-size: 1.8rem; color: #c0392b; filter: drop-shadow(0 0 8px rgba(192,57,43,.5)); animation: pulse-nav 4s ease infinite;}
@keyframes pulse-nav { 0%,100%{filter:drop-shadow(0 0 8px rgba(192,57,43,.4));} 50%{filter:drop-shadow(0 0 16px rgba(192,57,43,.8));} }
.nav-menu { flex: 1; display: flex; flex-direction: column; gap: 24px; width: 100%; }
.nav-item { display: flex; flex-direction: column; align-items: center; gap: 6px; color: #555; cursor: pointer; transition: .2s; }
.nav-item:hover { color: #888; }
.nav-item.active { color: #c0392b; }
.n-ic { font-size: 1.4rem; }
.n-txt { font-size: .65rem; font-weight: 600; letter-spacing: 1px; }
.nav-user { margin-top: auto; }
.btn-login-sm { width: 40px; height: 40px; border-radius: 12px; background: rgba(192,57,43,.1); border: 1px solid rgba(192,57,43,.3); color: #c0392b; cursor: pointer; font-weight: bold; transition: .2s; }
.btn-login-sm:hover { background: #c0392b; color: #fff; }
.user-av-wrap { width: 44px; height: 44px; border-radius: 12px; overflow: hidden; border: 2px solid transparent; cursor: pointer; transition: .2s; }
.user-av-wrap:hover { border-color: #c0392b; }
.nav-av { width: 100%; height: 100%; object-fit: cover; }
</style>
