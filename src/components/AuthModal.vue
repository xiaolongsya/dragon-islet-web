<template>
  <transition name="t-modal">
    <div class="modal-mask" @click.self="$emit('close')">
      <div class="modal-card">
        <div class="auth-title">{{ type==='login'?'唤醒龙魂':'建立契约' }}</div>
        <div class="auth-div"></div>
        <div class="form-g"><label class="f-lbl">名号</label><input class="f-inp" v-model="form.username" placeholder="名号"></div>
        <div class="form-g"><label class="f-lbl">密语</label><input class="f-inp" v-model="form.password" type="password" placeholder="密语"></div>
        <template v-if="type==='register'">
          <div class="form-g">
            <label class="f-lbl">手机验证</label>
            <div class="sms-row">
              <input class="f-inp sms-inp" v-model="form.phone" placeholder="手机号">
              <button class="btn-sms" @click="$emit('send-sms')" :disabled="smsCooldown>0">{{ smsCooldown>0?smsCooldown+'s':'获取验证码' }}</button>
            </div>
          </div>
          <div class="form-g"><label class="f-lbl">验证码</label><input class="f-inp" v-model="form.code" placeholder="6位验证码"></div>
        </template>
        <button class="btn-p f-btn" @click="$emit('submit')">{{ type==='login'?'登录':'注册' }}</button>
        <div class="auth-sw" @click="$emit('switch')">{{ type==='login'?'初入龙屿？建立契约':'已有契约？去登录' }}</div>
      </div>
    </div>
  </transition>
</template>

<script setup>
defineProps({
  type: String, // 'login' or 'register'
  form: Object,
  smsCooldown: Number
});
defineEmits(['close', 'submit', 'switch', 'send-sms']);
</script>

<style scoped>
.modal-mask { position: fixed; inset: 0; background: rgba(0,0,0,0.85); backdrop-filter: blur(8px); z-index: 1000; display: flex; align-items: center; justify-content: center; }
.modal-card { width: 90%; max-width: 400px; background: #0d0d0d; border: 1px solid rgba(255,255,255,0.08); border-radius: 24px; padding: 40px; box-shadow: 0 20px 50px rgba(0,0,0,0.5); }
.auth-title { font-family: 'Noto Serif SC', serif; font-size: 1.8rem; color: #f0f0f0; text-align: center; margin-bottom: 24px; letter-spacing: 4px; }
.auth-div { width: 40px; height: 3px; background: #c0392b; margin: 0 auto 32px; border-radius: 2px; }
.form-g { margin-bottom: 20px; }
.f-lbl { display: block; font-size: .75rem; color: #444; margin-bottom: 8px; letter-spacing: 2px; }
.f-inp { width: 100%; background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1); border-radius: 10px; padding: 12px 16px; color: #fff; outline: none; transition: .2s; }
.f-inp:focus { border-color: #c0392b; background: rgba(192,57,43,0.05); }
.sms-row { display: flex; gap: 10px; }
.sms-inp { flex: 1; }
.btn-sms { flex-shrink: 0; background: none; border: 1px solid rgba(255,255,255,0.1); color: #888; border-radius: 10px; padding: 0 16px; font-size: .8rem; cursor: pointer; transition: .2s; }
.btn-sms:hover:not(:disabled) { border-color: #c0392b; color: #c0392b; }
.btn-p { width: 100%; background: #c0392b; color: #fff; border: none; border-radius: 10px; padding: 14px; font-weight: bold; cursor: pointer; transition: .2s; margin-top: 10px; }
.btn-p:hover { background: #e74c3c; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(192,57,43,0.3); }
.auth-sw { text-align: center; margin-top: 24px; font-size: .85rem; color: #555; cursor: pointer; transition: .2s; }
.auth-sw:hover { color: #c0392b; }

.t-modal-enter-active, .t-modal-leave-active { transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.t-modal-enter-from, .t-modal-leave-to { opacity: 0; transform: scale(0.9) translateY(20px); }
</style>
