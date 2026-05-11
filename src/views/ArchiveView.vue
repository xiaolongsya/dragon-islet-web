<template>
  <div class="view-archives layout">
    <section class="chat-area">
      <div class="msg-list">
        <div class="my-msg-header">📜 龙屿史诗 · 纪元档案</div>
        <div v-if="archives.length===0" class="empty-tip">史官尚未落笔，岛屿尚在沉睡...</div>
        
        <div v-for="arc in archives" :key="arc.ID" class="archive-card">
          <div class="arc-header">
            <span class="arc-title">{{ arc.title }}</span>
            <span class="arc-date">{{ arc.date }}</span>
          </div>
          <div class="arc-content">{{ arc.content }}</div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
defineProps({
  archives: Array
});
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;}
.msg-list{flex:1;overflow-y:auto;padding:24px 48px;}
.my-msg-header { font-size: 1.2rem; font-weight: 600; color: #eee; margin-bottom: 32px; border-left: 4px solid #c0392b; padding-left: 12px; letter-spacing: 2px; }
.empty-tip { text-align: center; color: #333; padding: 100px 0; letter-spacing: 4px; }

.archive-card { background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.05); border-radius: 20px; padding: 32px; margin-bottom: 40px; transition: .3s; }
.archive-card:hover { border-color: rgba(192,57,43,0.3); background: rgba(255,255,255,0.03); }
.arc-header { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 24px; border-bottom: 1px solid rgba(255,255,255,0.03); padding-bottom: 16px; }
.arc-title { font-family: 'Noto Serif SC', serif; font-size: 1.4rem; color: #f0f0f0; letter-spacing: 2px; font-weight: 600; }
.arc-date { font-size: .85rem; color: #444; font-family: 'Inter'; }
.arc-content { font-size: 1.05rem; color: #bbb; line-height: 2; white-space: pre-wrap; letter-spacing: 1px; }
</style>
