<template>
  <transition name="t-modal">
    <div class="modal-mask" @click.self="$emit('close')">
      <div class="modal-card card-profile">
        <div class="pf-top">
          <label class="av-lg av-clickable">
            <img :src="editForm.avatar||user.avatar||defAv">
            <div class="av-overlay">幻化</div>
            <input type="file" hidden @change="$emit('upload-avatar', $event)" accept="image/*">
          </label>
          <div class="pf-name">{{ user.username }}</div>
          <div class="pf-sub">{{ user.title || '游侠 · 龙屿契约者' }}</div>
        </div>
        <div class="m-div"></div>
        <div class="form-g">
          <label class="f-lbl">易名</label>
          <input class="f-inp" v-model="editForm.username" placeholder="新的名号...">
        </div>
        <div class="form-g">
          <label class="f-lbl">游侠宣言</label>
          <textarea class="f-inp f-txt" v-model="editForm.motto" placeholder="留下一段你的现世宣言..."></textarea>
        </div>
        <button class="btn-p f-btn" @click="$emit('submit')">重塑契约</button>
        <button class="btn-ghost f-btn mt-s" @click="$emit('logout')">归隐山林 · 登出</button>
      </div>
    </div>
  </transition>
</template>

<script setup>
defineProps({
  user: Object,
  editForm: Object,
  defAv: String
});
defineEmits(['close', 'submit', 'logout', 'upload-avatar']);
</script>

<style scoped>
.modal-mask { position: fixed; inset: 0; background: rgba(0,0,0,0.85); backdrop-filter: blur(8px); z-index: 1000; display: flex; align-items: center; justify-content: center; }
.modal-card { width: 90%; max-width: 400px; background: #0d0d0d; border: 1px solid rgba(255,255,255,0.08); border-radius: 24px; padding: 40px; box-shadow: 0 20px 50px rgba(0,0,0,0.5); }
.pf-top { display: flex; flex-direction: column; align-items: center; margin-bottom: 24px; }
.av-lg { width: 100px; height: 100px; border-radius: 30px; overflow: hidden; position: relative; border: 2px solid rgba(192,57,43,0.3); margin-bottom: 16px; }
.av-lg img { width: 100%; height: 100%; object-fit: cover; }
.av-overlay { position: absolute; inset: 0; background: rgba(0,0,0,0.6); color: #fff; display: flex; align-items: center; justify-content: center; opacity: 0; transition: .2s; font-size: .8rem; }
.av-clickable:hover .av-overlay { opacity: 1; cursor: pointer; }
.pf-name { font-size: 1.4rem; font-weight: 600; color: #fff; margin-bottom: 4px; }
.pf-sub { font-size: .8rem; color: #444; letter-spacing: 2px; }
.m-div { height: 1px; background: rgba(255,255,255,0.05); margin-bottom: 24px; }
.form-g { margin-bottom: 20px; }
.f-lbl { display: block; font-size: .75rem; color: #444; margin-bottom: 8px; letter-spacing: 2px; }
.f-inp { width: 100%; background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1); border-radius: 10px; padding: 12px 16px; color: #fff; outline: none; transition: .2s; }
.f-inp:focus { border-color: #c0392b; background: rgba(192,57,43,0.05); }
.f-txt { height: 100px; resize: none; font-family: inherit; line-height: 1.6; }
.btn-p { width: 100%; background: #c0392b; color: #fff; border: none; border-radius: 10px; padding: 14px; font-weight: bold; cursor: pointer; transition: .2s; margin-top: 10px; }
.btn-p:hover { background: #e74c3c; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(192,57,43,0.3); }
.btn-ghost { background: none; border: 1px solid #333; color: #666; width: 100%; padding: 12px; border-radius: 10px; cursor: pointer; transition: .2s; font-size: .85rem; }
.btn-ghost:hover { border-color: #c0392b; color: #c0392b; }
.mt-s { margin-top: 12px; }

.t-modal-enter-active, .t-modal-leave-active { transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
.t-modal-enter-from, .t-modal-leave-to { opacity: 0; transform: scale(0.9) translateY(20px); }
</style>
