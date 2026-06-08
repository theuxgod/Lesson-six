<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'

type Theme = 'dark' | 'light'

const theme = ref<Theme>('dark')

const applyTheme = (value: Theme) => {
  document.documentElement.classList.toggle('light', value === 'light')
}

onMounted(() => {
  const saved = localStorage.getItem('theme') as Theme | null
  if (saved === 'light' || saved === 'dark') {
    theme.value = saved
  }
  applyTheme(theme.value)
})

watch(theme, (value) => {
  applyTheme(value)
  localStorage.setItem('theme', value)
})

const toggleTheme = () => {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
}

const links = [
  { label: 'Portfolio', href: 'https://example.com', icon: '🌐' },
  { label: 'Dribbble', href: 'https://dribbble.com', icon: '🏀' },
  { label: 'LinkedIn', href: 'https://linkedin.com', icon: '💼' },
  { label: 'Email', href: 'mailto:hello@example.com', icon: '✉️' },
]
</script>

<template>
  <main class="page">
    <button
      class="theme-toggle"
      type="button"
      :aria-label="theme === 'dark' ? 'Switch to light mode' : 'Switch to dark mode'"
      @click="toggleTheme"
    >
      {{ theme === 'dark' ? '☀️' : '🌙' }}
    </button>

    <section class="card">
      <div class="avatar" aria-hidden="true">BT</div>
      <h1 class="name">Brian Thomas</h1>
      <p class="tagline">Designer &amp; Developer · Crafting thoughtful digital experiences</p>

      <nav class="links" aria-label="Social links">
        <a
          v-for="link in links"
          :key="link.label"
          class="link-button"
          :href="link.href"
          target="_blank"
          rel="noopener noreferrer"
        >
          <span class="link-icon" aria-hidden="true">{{ link.icon }}</span>
          <span class="link-label">{{ link.label }}</span>
        </a>
      </nav>
    </section>
  </main>
</template>

<style scoped>
.page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem 1rem;
  position: relative;
}

.theme-toggle {
  position: absolute;
  top: 1rem;
  right: 1rem;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: 1px solid var(--border);
  background-color: var(--surface);
  color: var(--text);
  font-size: 1.1rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s ease, background-color 0.2s ease;
}

.theme-toggle:hover {
  background-color: var(--surface-hover);
  transform: rotate(15deg);
}

.theme-toggle:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.card {
  width: 100%;
  max-width: 480px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: 20px;
  padding: 2.5rem 1.75rem;
  box-shadow: var(--shadow);
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.avatar {
  width: 112px;
  height: 112px;
  border-radius: 50%;
  background: linear-gradient(135deg, #6c8cff 0%, #b06cff 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  font-weight: 700;
  font-size: 2rem;
  letter-spacing: 0.05em;
  margin-bottom: 1.25rem;
  box-shadow: 0 6px 18px rgba(108, 140, 255, 0.35);
}

.name {
  font-size: 1.75rem;
  font-weight: 700;
  margin: 0 0 0.5rem;
  letter-spacing: -0.01em;
}

.tagline {
  margin: 0 0 2rem;
  color: var(--text-muted);
  font-size: 0.95rem;
  font-weight: 300;
  line-height: 1.5;
}

.links {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.link-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.6rem;
  padding: 0.95rem 1.25rem;
  background-color: var(--surface-hover);
  color: var(--text);
  border: 1px solid var(--border);
  border-radius: 12px;
  text-decoration: none;
  font-size: 1rem;
  font-weight: 500;
  transition: transform 0.2s ease, background-color 0.2s ease, box-shadow 0.2s ease,
    border-color 0.2s ease;
}

.link-button:hover {
  transform: translateY(-2px);
  background-color: var(--bg);
  border-color: var(--accent);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
}

.link-button:active {
  transform: translateY(0);
}

.link-button:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.link-icon {
  font-size: 1.1rem;
}

@media (min-width: 600px) {
  .card {
    padding: 3rem 2.5rem;
  }

  .name {
    font-size: 2rem;
  }
}
</style>
