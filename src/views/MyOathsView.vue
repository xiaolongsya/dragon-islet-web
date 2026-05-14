<template>
  <div class="view-profile layout">
    <section class="chat-area">
      <div class="msg-list">
        <div class="profile-container">
          
          <!-- 第一部分：游侠通行证 (Header Card) -->
          <header class="profile-card pop-1">
            <div class="pf-identity">
              <div class="pf-avatar-wrap">
                <img :src="user.avatar || defAv" class="pf-avatar">
              </div>
              <div class="pf-main-info">
                <h1 class="pf-nickname">{{ user.username }}</h1>
                <div class="pf-badge-row">
                  <span class="pf-title">✦ {{ user.title || '初入龙屿的游侠' }}</span>
                  <span class="pf-exp">灵力值: {{ user.experience || 0 }}</span>
                  <button class="btn-edit-trigger" @click="$emit('open-profile-modal')">重塑契约</button>
                </div>
              </div>
            </div>
            <div class="pf-motto">
              <span class="motto-quote">“</span>
              {{ user.motto || '这个游侠很神秘，还没有留下任何宣言。' }}
            </div>
          </header>

          <!-- 第二部分：行纪内容 (Tabs / Sections) -->
          <div class="journal-sections">
            
            <!-- 历史誓言 -->
            <section class="journal-block pop-2">
              <div class="block-header">
                <h3>📜 往昔誓言 ({{ total }})</h3>
                <div class="block-line"></div>
              </div>
              
              <div v-if="items.length===0" class="empty-mini">尚未在龙屿留下任何誓约</div>
              <transition-group name="t-msg" tag="div" class="oath-list">
                <div v-for="msg in items" :key="msg.id" class="oath-item" :class="{'oath-recalled': msg.is_recalled}">
                  <div class="oath-meta">
                    <span class="oath-time">{{ fmtDate(msg.created_at) }}</span>
                    <span class="oath-tag" :class="msg.ai_interest?'tag-fire':'tag-void'">
                      {{ msg.ai_interest?'🔥 主的青睐':'🌪 凡言' }}
                    </span>
                  </div>
                  <div class="oath-content">
                    <span v-if="msg.is_recalled" class="txt-recalled">已在因果中抹除</span>
                    <span v-else>{{ msg.content }}</span>
                    <button v-if="!msg.is_recalled" class="btn-del-oath" @click="$emit('del-msg', msg.id)">✦</button>
                  </div>
                </div>
              </transition-group>

              <div class="pagination" v-if="total > limit">
                <button :disabled="page===1" @click="$emit('change-page', page-1)">←</button>
                <span class="pg-num">{{ page }} / {{ Math.ceil(total/limit) }}</span>
                <button :disabled="page >= Math.ceil(total/limit)" @click="$emit('change-page', page+1)">→</button>
              </div>
            </section>

            <!-- 鳞笺记录 -->
            <section class="journal-block pop-3">
              <div class="block-header">
                <h3>✉️ 鳞笺回响 ({{ feedbacks.length }})</h3>
                <div class="block-line"></div>
              </div>

              <div v-if="feedbacks.length===0" class="empty-mini">尚未向龙主投递过信笺</div>
              <div class="fb-journal">
                <div v-for="fb in feedbacks" :key="fb.id" class="fb-card">
                  <div class="fb-q-row">
                    <span class="fb-label">问</span>
                    <p class="fb-txt">{{ fb.content }}</p>
                    <button class="btn-fb-del" @click="$emit('del-fb', fb.id)">抹除</button>
                  </div>
                  <div class="fb-a-row">
                    <div v-if="fb.is_replied" class="fb-reply">
                      <span class="reply-head">主的回响:</span> {{ fb.reply_content }}
                    </div>
                    <div v-else class="fb-pending">静待云端回响...</div>
                  </div>
                  <div class="fb-footer">{{ fmtDate(fb.created_at) }}</div>
                </div>
              </div>
            </section>

          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
defineProps({
  items: Array,
  total: Number,
  page: Number,
  limit: Number,
  feedbacks: Array,
  user: Object
});
defineEmits(['change-page', 'del-msg', 'del-fb', 'open-profile-modal']);

const defAv = 'https://xiaolongya.cn/uploads/1778432333617872906.jpg';
const fmtDate = t => {
  const d = new Date(t);
  return `${d.getMonth()+1}/${d.getDate()} ${d.getHours()}:${String(d.getMinutes()).padStart(2, '0')}`;
};
</script>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;background: transparent;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;background: linear-gradient(to right, rgba(0,0,0,0.85) 0%, rgba(0,0,0,0.6) 50%, rgba(0,0,0,0.85) 100%);}
.msg-list{flex:1;overflow-y:auto;padding:60px 0;background: transparent;}

.profile-container { max-width: 1000px; margin: 0 auto; padding: 0 40px; }

/* Pass Card (Header) */
.profile-card { background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08); border-radius: 30px; padding: 40px; margin-bottom: 40px; backdrop-filter: blur(10px); box-shadow: 0 20px 50px rgba(0,0,0,0.5); }
.pf-identity { display: flex; align-items: center; gap: 32px; margin-bottom: 30px; }
.pf-avatar-wrap { width: 100px; height: 100px; border-radius: 28px; overflow: hidden; border: 2px solid #c0392b; box-shadow: 0 0 30px rgba(192,57,43,0.3); flex-shrink: 0; }
.pf-avatar { width: 100%; height: 100%; object-fit: cover; }
.pf-nickname { font-size: 2rem; color: #fff; letter-spacing: 4px; font-family: 'Noto Serif SC', serif; margin-bottom: 8px; }
.pf-badge-row { display: flex; gap: 16px; }
.pf-title { color: #c0392b; font-size: 0.9rem; font-weight: 600; letter-spacing: 2px; }
.pf-exp { color: #666; font-size: 0.85rem; border-left: 1px solid #333; padding-left: 16px; margin-right: auto; }
.btn-edit-trigger { background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #c0392b; font-size: 0.7rem; padding: 2px 10px; border-radius: 6px; cursor: pointer; transition: 0.2s; }
.btn-edit-trigger:hover { background: #c0392b; color: #fff; }
.pf-motto { font-size: 1.1rem; color: #aaa; line-height: 1.8; font-style: italic; position: relative; padding-left: 24px; }
.motto-quote { position: absolute; left: 0; top: -10px; font-size: 3rem; color: #c0392b; opacity: 0.3; font-family: serif; }

/* Journal Sections */
.journal-sections { display: grid; grid-template-columns: 1fr 1fr; gap: 40px; align-items: flex-start; }
.journal-block { background: rgba(20,20,20,0.8); border: 1px solid rgba(255,255,255,0.05); border-radius: 24px; padding: 32px; transform: translateZ(0); }
.block-header { margin-bottom: 24px; }
.block-header h3 { font-family: 'Noto Serif SC', serif; color: #eee; font-size: 1.2rem; margin-bottom: 12px; letter-spacing: 2px; }
.block-line { height: 1px; background: linear-gradient(to right, #c0392b, transparent); }

/* Oath List */
.oath-item { background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.03); border-radius: 12px; padding: 16px; margin-bottom: 16px; position: relative; transition: .3s; }
.oath-item:hover { border-color: rgba(192,57,43,0.3); background: rgba(255,255,255,0.04); }
.oath-meta { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.oath-time { font-size: 0.75rem; color: #444; }
.oath-tag { font-size: 0.65rem; padding: 2px 8px; border-radius: 4px; font-weight: 600; }
.tag-fire { color: #e74c3c; background: rgba(192,57,43,0.1); }
.tag-void { color: #444; }
.oath-content { color: #ccc; font-size: 0.95rem; line-height: 1.6; padding-right: 30px; }
.txt-recalled { color: #333; font-style: italic; }
.btn-del-oath { position: absolute; right: 12px; bottom: 12px; background: none; border: none; color: #222; cursor: pointer; transition: .2s; }
.btn-del-oath:hover { color: #c0392b; transform: scale(1.2); }

/* Feedback Cards */
.fb-card { background: rgba(255,255,255,0.02); border: 1px solid rgba(255,255,255,0.03); border-radius: 16px; padding: 20px; margin-bottom: 20px; }
.fb-q-row { display: flex; gap: 12px; margin-bottom: 16px; position: relative; }
.fb-label { background: #333; color: #888; font-size: 0.65rem; padding: 2px 6px; border-radius: 4px; height: fit-content; }
.fb-txt { color: #eee; font-size: 0.95rem; flex: 1; line-height: 1.6; }
.btn-fb-del { background: none; border: 1px solid #222; color: #444; font-size: 0.7rem; padding: 2px 6px; border-radius: 4px; cursor: pointer; height: fit-content; }
.btn-fb-del:hover { border-color: #c0392b; color: #c0392b; }
.fb-reply { background: rgba(192,57,43,0.05); border: 1px solid rgba(192,57,43,0.1); padding: 12px 16px; border-radius: 10px; color: #999; font-size: 0.9rem; line-height: 1.6; }
.reply-head { color: #c0392b; font-weight: bold; margin-right: 8px; }
.fb-pending { font-size: 0.8rem; color: #444; font-style: italic; }
.fb-footer { margin-top: 12px; font-size: 0.7rem; color: #333; text-align: right; }

/* Pagination */
.pagination { display: flex; align-items: center; justify-content: center; gap: 20px; margin-top: 24px; }
.pagination button { background: none; border: 1px solid #333; color: #666; width: 32px; height: 32px; border-radius: 8px; cursor: pointer; transition: .2s; }
.pagination button:hover:not(:disabled) { border-color: #c0392b; color: #c0392b; }
.pagination button:disabled { opacity: 0.2; }
.pg-num { font-size: 0.85rem; color: #444; }

.empty-mini { text-align: center; color: #222; padding: 40px 0; font-size: 0.9rem; letter-spacing: 2px; }

.pop-1 { animation: fade-up 0.6s ease both; }
.pop-2 { animation: fade-up 0.6s ease both 0.2s; }
.pop-3 { animation: fade-up 0.6s ease both 0.4s; }

@keyframes fade-up { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: none; } }

@media (max-width: 900px) {
  .journal-sections { grid-template-columns: 1fr; gap: 20px; }
  .profile-container { padding: 0 12px; width: 100%; box-sizing: border-box; }
  .pf-identity { flex-direction: column; text-align: center; gap: 15px; }
  .pf-avatar-wrap { margin: 0 auto; width: 70px; height: 70px; }
  .pf-nickname { font-size: 1.4rem; }
  .pf-badge-row { justify-content: center; flex-wrap: wrap; gap: 8px; font-size: 0.8rem; }
  .pf-exp { border-left: none; padding-left: 0; width: 100%; margin-top: 5px; opacity: 0.6; }
  .profile-card { padding: 25px 15px; border-radius: 20px; margin-bottom: 25px; width: 100%; box-sizing: border-box; }
  .journal-block { padding: 15px; border-radius: 20px; width: 100%; box-sizing: border-box; }
}
</style>
