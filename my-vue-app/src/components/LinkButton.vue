<script setup lang="ts">
defineProps<{
  label: string
  url: string
  icon: string
}>()
</script>

<template>
  <v-btn
    :href="url"
    :prepend-icon="icon"
    target="_blank"
    rel="noopener noreferrer"
    variant="tonal"
    size="large"
    block
    rounded="lg"
  >
    {{ label }}
  </v-btn>
</template>
