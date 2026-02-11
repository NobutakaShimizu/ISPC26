<template>
  <span ref="el" class="inline-math"></span>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import katex from 'katex'

const props = defineProps<{ formula: string }>()
const el = ref<HTMLSpanElement | null>(null)

function render() {
  if (!el.value || !props.formula) return
  katex.render(props.formula, el.value, { throwOnError: false, displayMode: false })
}

onMounted(render)
watch(() => props.formula, render)
</script>

<style scoped>
.inline-math {
  display: inline-block;
}
</style>
