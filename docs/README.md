---
pageLayout: page
---

<script setup>
import { ref, onMounted } from 'vue'
import PixelBlast from '@theme/background/PixelBlast.vue'

const hitokoto = ref('')
const githubStatus = ref('')
const githubEmoji = ref('🤗')

// 获取一言
const fetchHitokoto = async () => {
  try {
    const res = await fetch('https://v1.hitokoto.cn')
    const data = await res.json()
    hitokoto.value = data.hitokoto
  } catch {
    hitokoto.value = '只会写文档和挑毛病和纯Vibe Coding'
  }
}

// 获取GitHub状态
const fetchGithubStatus = async () => {
  try {
    const res = await fetch('https://api.github.com/users/LeafS825/events/public')
    const events = await res.json()
    if (events.length > 0) {
      const latest = events[0]
      const type = latest.type
      if (type === 'PushEvent') {
        githubStatus.value = '刚刚推送了代码'
        githubEmoji.value = '💻'
      } else if (type === 'CreateEvent') {
        githubStatus.value = '刚刚创建了仓库'
        githubEmoji.value = '🎉'
      } else if (type === 'IssuesEvent') {
        githubStatus.value = '刚刚提交了Issue'
        githubEmoji.value = '�'
      } else if (type === 'PullRequestEvent') {
        githubStatus.value = '刚刚提交了PR'
        githubEmoji.value = '🔀'
      } else {
        githubStatus.value = '刚刚在GitHub活动'
        githubEmoji.value = '🤗'
      }
    } else {
      githubStatus.value = '暂无最近活动'
      githubEmoji.value = '😴'
    }
  } catch {
    githubStatus.value = 'GitHub状态获取失败'
    githubEmoji.value = '😵'
  }
}

onMounted(() => {
  fetchHitokoto()
  fetchGithubStatus()
})
</script>

<div class="home-page">
  <!-- PixelBlast 背景效果 -->
  <div class="pixelblast-layer" aria-hidden="true">
    <ClientOnly>
      <PixelBlast
        class="pixelblast-canvas"
        variant="square"
        color="#5086a1"
        :speed="0.5"
        :pixel-size="4"
        :pattern-scale="2"
        :pattern-density="0.8"
        :edge-fade="0.5"
        :pixel-size-jitter="0"
        :antialias="true"
        :enable-ripples="true"
        :ripple-intensity-scale="1"
        :ripple-thickness="0.1"
        :ripple-speed="0.3"
        :liquid="false"
        :liquid-strength="0.1"
        :liquid-radius="1"
        :liquid-wobble-speed="4.5"
        :auto-pause-offscreen="true"
        :transparent="true"
        :noise-amount="0"
      />
    </ClientOnly>
  </div>

  <!-- 右上角一言 -->
  <div class="hitokoto-top">
    <p>{{ hitokoto }}</p>
  </div>

  <!-- 左下角名字 -->
  <div class="name-section">
    <div class="name-wrapper">
      <p class="name-prefix">我，</p>
      <h1 class="name-main">叶背影</h1>
    </div>
    <div class="github-status">
      <span class="status-emoji">{{ githubEmoji }}</span>
      <span class="status-text">{{ githubStatus }}</span>
    </div>
  </div>

  <!-- 右下角信息 -->
  <div class="info-section">
    <p class="role">高二学生 · SECTL 成员 & 贡献者</p>
    <div class="social-row">
      <a href="https://github.com/LeafS825" target="_blank" class="social-icon" title="GitHub">
        <Icon name="mdi:github" />
      </a>
      <a href="https://space.bilibili.com/1762621716" target="_blank" class="social-icon" title="Bilibili">
        <Icon name="mingcute:bilibili-line" />
      </a>
      <a href="mailto:871850079@qq.com" class="social-icon" title="Email">
        <Icon name="mdi:email-outline" />
      </a>
      <a href="/blog/" class="blog-btn">
        <Icon name="mdi:book-open-page-variant" />
        博客
      </a>
    </div>
  </div>
</div>

<style scoped>
.home-page {
  min-height: 100vh;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 2rem 3rem;
  background: var(--vp-c-bg);
  font-size: 1.25rem;
}

/* 右上角一言 */
.hitokoto-top {
  position: absolute;
  top: 2rem;
  right: 3rem;
  max-width: 400px;
  text-align: right;
}

.hitokoto-top p {
  margin: 0;
  font-size: 1.25rem;
  color: var(--vp-c-text-2);
  font-style: italic;
  opacity: 0.7;
}

/* 左下角名字区域 */
.name-section {
  position: absolute;
  bottom: 6rem;
  left: 3rem;
  display: flex;
  align-items: flex-end;
  gap: 1.5rem;
}

.name-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.name-prefix {
  font-size: 6rem;
  font-weight: 200;
  color: var(--vp-c-text-2);
  margin: 0;
  line-height: 1;
}

.name-main {
  font-size: 6rem;
  font-weight: 200;
  color: var(--vp-c-text-2);
  margin: 0;
  line-height: 1;
}

.github-status {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  margin-left: 0.5rem;
  padding: 0.75rem;
  background: var(--vp-c-bg-soft);
  border-radius: 16px;
  border: 1px solid var(--vp-c-divider);
}

.status-emoji {
  font-size: 2.25rem;
}

.status-text {
  font-size: 1rem;
  color: var(--vp-c-text-3);
  white-space: nowrap;
}

/* 右下角信息区域 */
.info-section {
  position: absolute;
  bottom: 6rem;
  right: 3rem;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 1.25rem;
}

.role {
  font-size: 1.35rem;
  color: var(--vp-c-text-2);
  margin: 0;
}

.social-row {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.social-icon {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-divider);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--vp-c-text-1);
  text-decoration: none;
  transition: all 0.25s ease;
  font-size: 1.75rem;
}

.social-icon:hover {
  border-color: var(--vp-c-brand-1);
  transform: scale(1.1);
}

.blog-btn {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1.5rem;
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-divider);
  border-radius: 28px;
  color: var(--vp-c-text-1);
  text-decoration: none;
  font-size: 1.15rem;
  transition: all 0.25s ease;
}

.blog-btn:hover {
  background: var(--vp-c-brand-1);
  border-color: var(--vp-c-brand-1);
  color: #fff;
}

/* PixelBlast 背景层 */
.pixelblast-layer {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: auto;
  overflow: hidden;
}

.pixelblast-layer :deep(.home-hero-effect-pixel-blast) {
  position: absolute;
  inset: 0;
  pointer-events: auto;
}

/* 确保内容在背景之上 */
.hitokoto-top,
.name-section,
.info-section {
  z-index: 1;
}

/* 响应式 */
@media screen and (max-width: 768px) {
  .home-page {
    padding: 1rem;
    justify-content: flex-end;
    min-height: calc(100dvh - 64px);
    box-sizing: border-box;
  }

  .hitokoto-top {
    position: absolute;
    top: 1rem;
    left: 1rem;
    right: 1rem;
    text-align: center;
    margin-bottom: 0;
    max-width: 100%;
  }

  .name-section {
    position: static;
    flex-direction: column;
    align-items: flex-start;
    justify-content: flex-start;
    margin-bottom: 0.25rem;
    width: 100%;
    flex-shrink: 0;
  }

  .name-wrapper {
    flex-direction: column;
  }

  .name-prefix {
    font-size: 3rem;
  }

  .name-main {
    font-size: 3rem;
  }

  .github-status {
    flex-direction: row;
    margin-left: 0;
    margin-top: 0.25rem;
    flex-shrink: 0;
  }

  .info-section {
    position: static;
    align-items: flex-start;
    margin-top: 0;
    padding-bottom: 0.5rem;
    gap: 0.25rem;
    width: 100%;
    flex-shrink: 0;
  }

  .social-row {
    flex-wrap: wrap;
    justify-content: flex-start;
    gap: 0.5rem;
  }
}
</style>
