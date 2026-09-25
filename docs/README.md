---
pageLayout: page
backToTop: false
---

<script setup>
import { onMounted, ref } from 'vue'

const FALLBACK_QUOTE = '只会写文档和挑毛病和纯 Vibe Coding'

const quote = ref(FALLBACK_QUOTE)
const ghVisible = ref(false)
const ghEmoji = ref('💻')
const ghText = ref('')
const embers = ref([])
const bgImg = ref(null)

/* ---------------- 一言 ---------------- */
const loadQuote = async () => {
  try {
    const ctrl = new AbortController()
    const timer = setTimeout(() => ctrl.abort(), 4000)
    const res = await fetch('https://v1.hitokoto.cn', { signal: ctrl.signal })
    clearTimeout(timer)
    const data = await res.json()
    if (data && data.hitokoto) quote.value = data.hitokoto
  } catch {
    quote.value = FALLBACK_QUOTE
  }
}

/* ---------------- GitHub 动态（本地缓存，失败静默隐藏） ---------------- */

const GH_CACHE_KEY = 'leafs-gh-activity'
const GH_TTL = 10 * 60 * 1000

const relTime = (iso) => {
  const min = Math.floor((Date.now() - new Date(iso).getTime()) / 60000)
  if (min < 1) return '刚刚'
  if (min < 60) return `${min} 分钟前`
  const hour = Math.floor(min / 60)
  if (hour < 24) return `${hour} 小时前`
  return `${Math.floor(hour / 24)} 天前`
}

const describeEvent = (e) => {
  const repo = (e.repo && e.repo.name) || ''
  const at = relTime(e.created_at)
  switch (e.type) {
    case 'PushEvent': {
      const n = (e.payload && e.payload.size) || 1
      return { emoji: '💻', text: `${at}推送了 ${n} 个提交到 ${repo}` }
    }
    case 'CreateEvent':
      return { emoji: '🎉', text: `${at}创建了仓库 ${repo}` }
    case 'IssuesEvent':
      return { emoji: '📝', text: `${at}提交了 Issue（${repo}）` }
    case 'PullRequestEvent':
      return { emoji: '🔀', text: `${at}提交了 PR（${repo}）` }
    case 'ReleaseEvent':
      return { emoji: '🚀', text: `${at}发布了新版本（${repo}）` }
    default:
      return { emoji: '🤗', text: `${at}在 GitHub 活动（${repo}）` }
  }
}

const renderGh = (event) => {
  const { emoji, text } = describeEvent(event)
  ghEmoji.value = emoji
  ghText.value = text
  ghVisible.value = true
}

const loadGithub = async () => {
  try {
    const raw = localStorage.getItem(GH_CACHE_KEY)
    if (raw) {
      const cached = JSON.parse(raw)
      if (Date.now() - cached.ts < GH_TTL && Array.isArray(cached.events)) {
        if (cached.events.length) renderGh(cached.events[0])
        return
      }
    }
  } catch { /* 缓存损坏则重新请求 */ }

  try {
    const ctrl = new AbortController()
    const timer = setTimeout(() => ctrl.abort(), 5000)
    const res = await fetch('https://api.github.com/users/LeafS825/events/public', {
      signal: ctrl.signal,
      headers: { Accept: 'application/vnd.github+json' },
    })
    clearTimeout(timer)
    if (!res.ok) throw new Error(String(res.status))
    const events = await res.json()
    localStorage.setItem(GH_CACHE_KEY, JSON.stringify({ ts: Date.now(), events }))
    if (Array.isArray(events) && events.length) renderGh(events[0])
  } catch {
    ghVisible.value = false
  }
}

/* ---------------- 背景视差 ---------------- */

const bindParallax = () => {
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return
  if (!window.matchMedia('(pointer: fine)').matches) return
  window.addEventListener('mousemove', (ev) => {
    if (!bgImg.value) return
    const dx = (ev.clientX / window.innerWidth - 0.5) * -24
    const dy = (ev.clientY / window.innerHeight - 0.5) * -16
    bgImg.value.style.setProperty('--px', `${dx.toFixed(1)}px`)
    bgImg.value.style.setProperty('--py', `${dy.toFixed(1)}px`)
  })
}

/* ---------------- 火星粒子 ---------------- */

const spawnEmbers = () => {
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return
  const list = []
  for (let i = 0; i < 18; i++) {
    list.push({
      left: Math.random() * 100,
      size: 3 + Math.random() * 4,
      dur: 11 + Math.random() * 14,
      delay: -Math.random() * 20,
      drift: (Math.random() - 0.5) * 120,
    })
  }
  embers.value = list
}

onMounted(() => {
  loadQuote()
  loadGithub()
  bindParallax()
  spawnEmbers()
})
</script>

<div class="home-page">
  <!-- 全屏插画背景 -->
  <div class="bg" aria-hidden="true">
    <img ref="bgImg" class="bg-img" src="/home-bg.jpg" alt="">
    <div class="bg-scrim"></div>
  </div>

  <!-- 火星粒子层 -->
  <div class="embers" aria-hidden="true">
    <span
      v-for="(e, i) in embers"
      :key="i"
      class="ember"
      :style="{
        left: e.left + '%',
        width: e.size + 'px',
        height: e.size + 'px',
        animationDuration: e.dur + 's',
        animationDelay: e.delay + 's',
        '--drift': e.drift + 'px',
      }"
    ></span>
  </div>

  <!-- 左下磨砂玻璃卡片 -->
  <section class="hero-card">
    <p class="quote"><span>{{ quote }}</span></p>
    <p class="prefix">我，</p>
    <h1 class="name">叶背影</h1>
    <p class="role">高二学生 · SECTL 成员 &amp; 贡献者</p>
    <div class="gh-pill" v-show="ghVisible">
      <span class="gh-dot"></span>
      <span class="gh-emoji">{{ ghEmoji }}</span>
      <span class="gh-text">{{ ghText }}</span>
    </div>
    <div class="actions">
      <div class="socials">
        <a class="icon-btn" href="https://github.com/LeafS825" target="_blank" rel="noopener" title="GitHub" aria-label="GitHub">
          <svg viewBox="0 0 16 16" fill="currentColor" aria-hidden="true"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27s1.36.09 2 .27c1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.01 8.01 0 0 0 16 8c0-4.42-3.58-8-8-8z"/></svg>
        </a>
        <a class="icon-btn" href="https://space.bilibili.com/1762621716" target="_blank" rel="noopener" title="Bilibili" aria-label="Bilibili">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" aria-hidden="true"><path d="M7 3.5 9.5 6M17 3.5 14.5 6"/><rect x="2.5" y="6" width="19" height="14.5" rx="4.5"/><path d="M7.8 11.2v2.4M16.2 11.2v2.4"/></svg>
        </a>
        <a class="icon-btn" href="mailto:871850079@qq.com" title="Email" aria-label="Email">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="2.5" y="5" width="19" height="14" rx="3"/><path d="m4.2 7.6 7.8 5.4 7.8-5.4"/></svg>
        </a>
      </div>
      <a class="btn-primary" href="/blog/">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M4 5.5h6a2.5 2.5 0 0 1 2.5 2.5v11A2 2 0 0 0 10.5 17.5H4zM20 5.5h-6a2.5 2.5 0 0 0-2.5 2.5v11a2 2 0 0 1 2-2H20z"/></svg>
        进入博客
      </a>
    </div>
  </section>

  <!-- 右缘竖排签名 -->
  <aside class="side-type" aria-hidden="true">叶随风动，背影由心；不问归期，只记途经</aside>
</div>

<style scoped>
/* ============================================================
   首页：全屏插画 KV + 磨砂玻璃 UI
   ============================================================ */

.home-page {
  position: relative;
  /* 恰好占满一屏：预留页脚（及窄屏时在文档流内的导航）高度，避免首屏出现滚动条 */
  min-height: calc(100vh - var(--vp-footer-height, 70px) - var(--vp-nav-height, 64px));
  background: var(--vp-c-bg);
}

@media (min-width: 960px) {
  /* 导航 fixed 不占文档流 */
  .home-page {
    min-height: calc(100vh - var(--vp-footer-height, 70px));
  }
}

/* ---------------- 背景 ---------------- */

.bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  overflow: hidden;
  animation: breathe 32s ease-in-out infinite alternate;
  pointer-events: none;
}

.bg-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center 28%;
  transform: translate3d(var(--px, 0), var(--py, 0), 0) scale(1.06);
  transition: transform 0.9s cubic-bezier(0.22, 0.61, 0.36, 1);
  will-change: transform;
}

@keyframes breathe {
  from { transform: scale(1); }
  to   { transform: scale(1.05); }
}

.bg-scrim {
  position: absolute;
  inset: 0;
  background:
    linear-gradient(180deg, rgba(12, 18, 44, 0.34) 0%, rgba(12, 18, 44, 0) 20%),
    linear-gradient(78deg, rgba(24, 32, 72, 0.22) 0%, rgba(24, 32, 72, 0) 42%);
  pointer-events: none;
}

/* ---------------- 火星粒子 ---------------- */

.embers {
  position: fixed;
  inset: 0;
  z-index: 0;
  overflow: hidden;
  pointer-events: none;
}

.ember {
  position: absolute;
  bottom: -12px;
  border-radius: 50%;
  background: radial-gradient(circle, #ffffff 0%, rgba(255, 255, 255, 0) 70%);
  box-shadow: 0 0 8px 2px rgba(255, 255, 255, 0.55);
  opacity: 0;
  animation: rise linear infinite;
}

@keyframes rise {
  0%   { transform: translate3d(0, 0, 0) scale(0.7); opacity: 0; }
  12%  { opacity: 0.9; }
  70%  { opacity: 0.55; }
  100% { transform: translate3d(var(--drift, 30px), -105vh, 0) scale(1.1); opacity: 0; }
}

/* ---------------- 磨砂玻璃卡片 ---------------- */

.hero-card {
  position: fixed;
  left: clamp(20px, 3vw, 56px);
  bottom: clamp(56px, 8vh, 88px);
  z-index: 1;
  width: min(440px, calc(100vw - 40px));
  padding: 2rem 2.2rem 2.1rem;
  border-radius: 28px;
  border: 1px solid rgba(255, 255, 255, 0.55);
  backdrop-filter: blur(24px) saturate(1.5);
  -webkit-backdrop-filter: blur(24px) saturate(1.5);
  box-shadow: 0 26px 70px rgba(26, 36, 88, 0.22);
}

.hero-card > * {
  animation: rise-in 0.9s cubic-bezier(0.22, 0.61, 0.36, 1) both;
}
.hero-card > *:nth-child(1) { animation-delay: 0.10s; }
.hero-card > *:nth-child(2) { animation-delay: 0.22s; }
.hero-card > *:nth-child(3) { animation-delay: 0.32s; }
.hero-card > *:nth-child(4) { animation-delay: 0.42s; }
.hero-card > *:nth-child(5) { animation-delay: 0.52s; }
.hero-card > *:nth-child(6) { animation-delay: 0.62s; }

@keyframes rise-in {
  from { opacity: 0; transform: translateY(18px); }
  to   { opacity: 1; transform: translateY(0); }
}

.quote {
  display: flex;
  align-items: center; /* 单行时在预留高度内垂直居中 */
  margin: 0 0 1.5rem;
  min-height: 2.8em;
  font-size: 0.95rem;
  line-height: 1.7;
  font-style: italic;
  color: #5b678a;
  border-left: 3px solid #1fa89e;
  padding-left: 0.85rem;
}

.quote span::before { content: '「'; }
.quote span::after  { content: '」'; }

.prefix {
  margin: 0;
  font-size: 1.3rem;
  font-weight: 300;
  color: #5b678a;
  letter-spacing: 0.08em;
}

.name {
  margin: 0.1em 0 0.4em;
  font-family: 'CustomFont', Georgia, 'Songti SC', 'Noto Serif SC', serif;
  font-size: clamp(3rem, 5.5vw, 4.4rem);
  font-weight: 500;
  line-height: 1.04;
  letter-spacing: 0.06em;
  color: #22304f;
}

.role {
  margin: 0;
  font-size: 1rem;
  letter-spacing: 0.04em;
  color: #5b678a;
}

.gh-pill {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 1.1rem;
  padding: 0.42rem 0.9rem;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.35);
  border: 1px solid rgba(255, 255, 255, 0.6);
  font-size: 0.85rem;
  color: #5b678a;
}

.gh-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #1fa89e;
  box-shadow: 0 0 0 3px rgba(31, 168, 158, 0.2);
}

.gh-emoji { font-size: 1rem; }

.actions {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.7rem;
  margin-top: 1.5rem;
}

.socials {
  display: flex;
  gap: 0.55rem;
}

.icon-btn {
  display: grid;
  place-items: center;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.66);
  border: 1px solid rgba(255, 255, 255, 0.92);
  color: #22304f;
  transition: transform 0.25s ease, border-color 0.25s ease, color 0.25s ease,
    box-shadow 0.25s ease;
}

.icon-btn svg { width: 20px; height: 20px; }

.icon-btn:hover {
  transform: translateY(-3px);
  border-color: #1fa89e;
  color: #1fa89e;
  box-shadow: 0 8px 20px rgba(31, 168, 158, 0.22);
}

.btn-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.55rem;
  padding: 0.78rem 1.6rem;
  border-radius: 999px;
  background: linear-gradient(135deg, #1fa89e, #4a8fe0);
  color: #fff;
  text-decoration: none;
  font-size: 1rem;
  letter-spacing: 0.06em;
  box-shadow: 0 12px 28px rgba(31, 142, 178, 0.36);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}

.btn-primary svg { width: 18px; height: 18px; }

.btn-primary:hover {
  transform: translateY(-3px);
  box-shadow: 0 16px 34px rgba(31, 142, 178, 0.46);
}

/* ---------------- 右缘竖排签名 ---------------- */

.side-type {
  position: fixed;
  right: 26px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 1;
  writing-mode: vertical-rl;
  font-size: 0.92rem;
  font-weight: 300;
  letter-spacing: 0.38em;
  color: rgba(255, 255, 255, 0.88);
  text-shadow: 0 1px 12px rgba(18, 26, 64, 0.55);
  pointer-events: none;
  user-select: none;
}

/* ---------------- 响应式 ---------------- */

@media (max-width: 1100px) {
  .side-type { display: none; }
}

/* 竖排整句约 370px 高，窗口太矮时隐藏以免溢出 */
@media (max-height: 480px) {
  .side-type { display: none; }
}

@media (max-width: 640px) {
  /* 水平方向始终居中裁切，角色保持在屏幕正中央 */
  .bg-img { object-position: center 30%; }

  .hero-card {
    left: 16px;
    right: 16px;
    bottom: 64px;
    width: auto;
    padding: 1.5rem 1.4rem 1.6rem;
  }

  .name { font-size: clamp(2.6rem, 13vw, 3.4rem); }
}

@media (max-height: 640px) and (min-width: 641px) {
  .hero-card { bottom: 40px; padding: 1.4rem 1.7rem; }
  .quote { margin-bottom: 0.9rem; }
}

/* ---------------- 动效偏好 ---------------- */

@media (prefers-reduced-motion: reduce) {
  .bg { animation: none; }
  .bg-img { transition: none; transform: scale(1.02); }
  .ember { display: none; }
  .hero-card > * { animation: none; }
}
</style>
