<script setup lang="ts">
import { computed, onMounted, watch } from 'vue'
import { useTheme } from 'vuetify'
import LinkButton from '@/components/LinkButton.vue'
import WeatherWidget from '@/components/WeatherWidget.vue'

const theme = useTheme()

const isDark = computed(() => theme.global.name.value === 'dark')

const toggleTheme = () => {
  theme.change(isDark.value ? 'light' : 'dark')
}

onMounted(() => {
  const saved = localStorage.getItem('theme')
  if (saved === 'light' || saved === 'dark') {
    theme.change(saved)
  }
})

watch(
  () => theme.global.name.value,
  (value) => {
    localStorage.setItem('theme', value)
  },
)

const links = [
  { label: 'Portfolio', href: 'https://example.com', icon: 'mdi-web' },
  { label: 'Dribbble', href: 'https://dribbble.com', icon: 'mdi-basketball' },
  { label: 'LinkedIn', href: 'https://linkedin.com', icon: 'mdi-linkedin' },
  { label: 'Email', href: 'mailto:hello@example.com', icon: 'mdi-email-outline' },
]
</script>

<template>
  <v-container class="fill-height pa-4" fluid>
    <v-btn
      class="theme-toggle"
      :icon="isDark ? 'mdi-weather-sunny' : 'mdi-weather-night'"
      variant="tonal"
      size="small"
      :aria-label="isDark ? 'Switch to light mode' : 'Switch to dark mode'"
      @click="toggleTheme"
    />

    <v-row justify="center" align="center" class="fill-height w-100" no-gutters>
      <v-col cols="12" sm="10" md="6" lg="5" xl="4" class="d-flex justify-center">
        <v-card class="pa-6 pa-sm-8 text-center w-100" max-width="480" rounded="xl" elevation="8">
          <v-avatar size="112" color="primary" class="mb-4 mx-auto">
            <span class="text-h5 font-weight-bold">BT</span>
          </v-avatar>

          <h1 class="text-h5 text-sm-h4 font-weight-bold mb-2">Brian Thomas</h1>
          <p class="text-body-2 text-medium-emphasis mb-6">
            Designer &amp; Developer · Crafting thoughtful digital experiences
          </p>

          <div class="d-flex flex-column ga-3">
            <LinkButton
              v-for="link in links"
              :key="link.label"
              :label="link.label"
              :url="link.href"
              :icon="link.icon"
            />
          </div>

          <WeatherWidget class="mt-6" />
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
.theme-toggle {
  position: absolute;
  top: 1rem;
  right: 1rem;
  z-index: 1;
}
</style>
