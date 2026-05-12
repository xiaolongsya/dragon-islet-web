<template>
  <div class="view-about layout">
    <section class="chat-area">
      <div class="msg-list">
        <div class="about-container">
          <header class="about-header">
            <div class="origin-logo-wrap">
              <img src="https://xiaolongya.cn/uploads/1778432390917936196.jpg" class="origin-logo">
            </div>
            <h1 class="origin-title">小龙 · 龙屿之主</h1>
            <p class="origin-motto">“山重水复疑无路，柳暗花明又一村”</p>
          </header>

          <div class="about-content">
            <section class="about-section pop-1">
              <h2 class="sec-title">✦ 起源故事</h2>
              <div class="sec-body">
                <p>因热爱而生，因创造而立，因分享而远。</p>
                <p>作为这片数字领地的守护者，我终日穿梭于逻辑的迷宫与创意的旷野，坚信每一行代码都是通往无限可能的咒语。在这里，我构建，我记录，我与每一个到访的游侠分享这份对技术最纯粹的热情。</p>
              </div>
            </section>

            <section class="about-section pop-2">
              <h2 class="sec-title">✦ 秘法修为</h2>
              <div class="skill-groups">
                <div class="skill-category">
                  <h3>御火术 (Backend Arts)</h3>
                  <div class="skill-tags">
                    <span>Java</span><span>SpringBoot</span><span>MyBatis</span>
                    <span>Golang</span><span>Gin</span><span>GORM</span>
                  </div>
                </div>
                <div class="skill-category">
                  <h3>幻影阵 (Frontend Arts)</h3>
                  <div class="skill-tags">
                    <span>Vue</span><span>Vite</span><span>CSS Magic</span>
                  </div>
                </div>
                <div class="skill-category">
                  <h3>唤灵术 (AI Agents)</h3>
                  <div class="skill-tags ai-tags">
                    <span>Antigravity</span><span>Cursor</span><span>Copilot</span>
                    <span>OpenClaw</span><span>Hermes</span>
                  </div>
                </div>
                <div class="skill-category">
                  <h3>磐石阵 (Data & Ops)</h3>
                  <div class="skill-tags">
                    <span>MySQL</span><span>Redis</span><span>1Panel</span><span>Deploy</span>
                  </div>
                </div>
              </div>
            </section>

            <section class="about-section pop-3">
              <h2 class="sec-title">✦ 代码遗迹 (Chronicles)</h2>
              <div class="repo-grid">
                <a href="https://github.com/xiaolongsya/dragon-islet" target="_blank" class="repo-card">
                  <div class="repo-type">BACKEND</div>
                  <div class="repo-name">dragon-islet</div>
                  <div class="repo-desc">Go + Gin + GORM 核心驱动</div>
                </a>
                <a href="https://github.com/xiaolongsya/dragon-islet-web" target="_blank" class="repo-card">
                  <div class="repo-type">FRONTEND</div>
                  <div class="repo-name">dragon-islet-web</div>
                  <div class="repo-desc">Vue 3 + Vite 幻影界面</div>
                </a>
              </div>
            </section>

            <section class="about-section pop-4">
              <h2 class="sec-title">✦ 时空锚点</h2>
              <div class="links-row">
                <a href="https://github.com/xiaolongsya" target="_blank" class="link-btn">
                  <span class="l-ic">🐙</span> GitHub 主轴
                </a>
              </div>
            </section>
          </div>

          <footer class="about-footer">
            <p>“山川是不卷的收容，巨龙是不熄的灯火。”</p>
          </footer>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.layout{flex:1;display:flex;overflow:hidden;background: transparent;}
.chat-area{flex:1;display:flex;flex-direction:column;overflow:hidden;min-width:0;background: linear-gradient(to right, rgba(0,0,0,0.85) 0%, rgba(0,0,0,0.6) 50%, rgba(0,0,0,0.85) 100%);}
.msg-list{flex:1;overflow-y:auto;padding:60px 0;background: transparent;}

.about-container { max-width: 900px; margin: 0 auto; padding: 0 40px; }
.about-header { text-align: center; margin-bottom: 80px; animation: fade-down 0.8s ease both; }
.origin-logo-wrap { width: 140px; height: 140px; margin: 0 auto 28px; border-radius: 40px; overflow: hidden; border: 2px solid #c0392b; box-shadow: 0 0 40px rgba(192,57,43,0.4); transform: rotate(-3deg); transition: 0.5s; }
.origin-logo-wrap:hover { transform: rotate(0) scale(1.05); }
.origin-logo { width: 100%; height: 100%; object-fit: cover; }
.origin-title { font-size: 2.5rem; color: #f0f0f0; letter-spacing: 8px; margin-bottom: 12px; font-family: 'Noto Serif SC', serif; }
.origin-motto { color: #888; font-style: italic; letter-spacing: 2px; }

.about-section { margin-bottom: 50px; background: rgba(20,20,20,0.8); padding: 40px; border-radius: 28px; border: 1px solid rgba(255,255,255,0.06); box-shadow: 0 20px 40px rgba(0,0,0,0.4); transform: translateZ(0); }
.sec-title { font-size: 1.4rem; color: #c0392b; margin-bottom: 28px; letter-spacing: 3px; font-family: 'Noto Serif SC', serif; border-bottom: 1px solid rgba(192,57,43,0.2); padding-bottom: 12px; display: inline-block;}
.sec-body p { font-size: 1.1rem; line-height: 2; color: #bbb; margin-bottom: 16px; }

.skill-groups { display: grid; grid-template-columns: 1fr 1fr; gap: 32px; }
.skill-category h3 { font-size: 1rem; color: #eee; margin-bottom: 16px; opacity: 0.8; letter-spacing: 1px; }
.skill-tags { display: flex; flex-wrap: wrap; gap: 10px; }
.skill-tags span { background: rgba(255,255,255,0.05); padding: 6px 14px; border-radius: 8px; font-size: 0.9rem; color: #aaa; border: 1px solid rgba(255,255,255,0.05); transition: 0.3s; }
.skill-tags span:hover { border-color: #c0392b; color: #fff; background: rgba(192,57,43,0.1); }
.ai-tags span { border-color: rgba(192,57,43,0.3); color: #ff6b6b; }

.repo-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
.repo-card { background: rgba(0,0,0,0.3); padding: 24px; border-radius: 16px; border: 1px solid rgba(255,255,255,0.05); text-decoration: none; transition: 0.4s; }
.repo-card:hover { transform: translateY(-5px); border-color: #c0392b; background: rgba(192,57,43,0.05); }
.repo-type { font-size: 0.7rem; font-weight: 800; color: #c0392b; margin-bottom: 8px; letter-spacing: 2px; }
.repo-name { font-size: 1.2rem; color: #eee; margin-bottom: 8px; font-weight: 600; }
.repo-desc { font-size: 0.9rem; color: #666; }

.links-row { display: flex; gap: 20px; justify-content: center; }
.link-btn { display: flex; align-items: center; gap: 12px; background: rgba(192,57,43,0.1); border: 1px solid rgba(192,57,43,0.3); color: #eee; padding: 14px 36px; border-radius: 14px; text-decoration: none; transition: 0.3s; font-weight: 600; }
.link-btn:hover { background: #c0392b; border-color: #ff4d4d; transform: scale(1.05); box-shadow: 0 0 30px rgba(192,57,43,0.3); }

.about-footer { text-align: center; padding: 60px 0; color: #333; font-style: italic; letter-spacing: 4px; font-size: 0.9rem; }

.pop-1 { animation: fade-up 0.6s ease both 0.1s; }
.pop-2 { animation: fade-up 0.6s ease both 0.2s; }
.pop-3 { animation: fade-up 0.6s ease both 0.3s; }
.pop-4 { animation: fade-up 0.6s ease both 0.4s; }

@keyframes fade-down { from { opacity: 0; transform: translateY(-30px); } to { opacity: 1; transform: none; } }
@keyframes fade-up { from { opacity: 0; transform: translateY(30px); } to { opacity: 1; transform: none; } }

@media (max-width: 768px) {
  .skill-groups, .repo-grid { grid-template-columns: 1fr; }
  .about-container { padding: 0 24px; }
  .origin-title { font-size: 2rem; }
}
</style>
