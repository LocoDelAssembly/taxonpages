<template>
  <slot v-if="show"></slot>
  <div v-else></div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const show = ref(false)

onMounted(() => {
  show.value = true
})
</script>
