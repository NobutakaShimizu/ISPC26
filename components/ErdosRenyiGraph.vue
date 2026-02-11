<template>
  <div class="er-graph-wrap">
    <canvas ref="canvas" :width="size" :height="size" class="er-canvas"></canvas>
    <div class="er-buttons">
      <button type="button" class="er-btn er-btn-pc" @click="plantClique">PCにする</button>
      <button type="button" class="er-btn er-btn-restore" @click="restore">元に戻す</button>
    </div>
    <p class="er-caption">{{ caption }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

const props = withDefaults(
  defineProps<{
    n?: number
    p?: number
    seed?: number
    size?: number
    cliqueSize?: number
    caption?: string
  }>(),
  {
    n: 100,
    p: 0.06,
    seed: 42,
    size: 420,
    cliqueSize: 10,
    caption: 'ER(100, 0.06) の例（100頂点）。可視性のため辺確率を 0.06 としている（定義では 1/2）。',
  }
)

const canvas = ref<HTMLCanvasElement | null>(null)
const baseEdges = ref<[number, number][]>([])
const plantedClique = ref<number[] | null>(null)
const highlightClique = ref(false)
/** 1 = 赤, 0 = 青。0〜1 でグラデーション */
const highlightBlend = ref(0)
let highlightTimeout: ReturnType<typeof setTimeout> | null = null
let transitionStartTime = 0
const TRANSITION_DURATION_MS = 800
let animationFrameId: number | null = null

// シード付き簡易乱数 (Mulberry32)
function mulberry32(seed: number) {
  return function () {
    let t = (seed += 0x6d2b79f5)
    t = Math.imul(t ^ (t >>> 15), t | 1)
    t ^= t + Math.imul(t ^ (t >>> 7), t | 61)
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296
  }
}

function buildERGraph(n: number, p: number, seed: number) {
  const rng = mulberry32(seed)
  const edges: [number, number][] = []
  for (let i = 0; i < n; i++) {
    for (let j = i + 1; j < n; j++) {
      if (rng() < p) edges.push([i, j])
    }
  }
  return edges
}

// Fisher-Yates でランダムに k 個の頂点を選択
function pickRandomVertices(n: number, k: number): number[] {
  const arr = Array.from({ length: n }, (_, i) => i)
  for (let i = 0; i < k; i++) {
    const j = i + Math.floor(Math.random() * (n - i))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr.slice(0, k)
}

function stopTransition() {
  if (animationFrameId != null) {
    cancelAnimationFrame(animationFrameId)
    animationFrameId = null
  }
  highlightBlend.value = 0
}

function runTransition() {
  transitionStartTime = performance.now()
  function tick(t: number) {
    const elapsed = t - transitionStartTime
    if (elapsed >= TRANSITION_DURATION_MS) {
      highlightBlend.value = 0
      animationFrameId = null
      draw()
      return
    }
    highlightBlend.value = 1 - elapsed / TRANSITION_DURATION_MS
    draw()
    animationFrameId = requestAnimationFrame(tick)
  }
  animationFrameId = requestAnimationFrame(tick)
}

function plantClique() {
  if (highlightTimeout) clearTimeout(highlightTimeout)
  stopTransition()
  plantedClique.value = pickRandomVertices(props.n, props.cliqueSize)
  highlightClique.value = true
  highlightBlend.value = 1
  draw()
  highlightTimeout = setTimeout(() => {
    highlightClique.value = false
    highlightTimeout = null
    runTransition()
  }, 1000)
}

function restore() {
  if (highlightTimeout) {
    clearTimeout(highlightTimeout)
    highlightTimeout = null
  }
  stopTransition()
  plantedClique.value = null
  highlightClique.value = false
  highlightBlend.value = 0
  draw()
}

function initBaseEdges() {
  baseEdges.value = buildERGraph(props.n, props.p, props.seed)
}

function draw() {
  if (!canvas.value) return
  const ctx = canvas.value.getContext('2d')
  if (!ctx) return

  const { n, size } = props
  const center = size / 2
  const radius = (size / 2) * 0.88
  const nodeRadius = 2.5

  const positions: { x: number; y: number }[] = []
  for (let i = 0; i < n; i++) {
    const angle = (2 * Math.PI * i) / n - Math.PI / 2
    positions.push({
      x: center + radius * Math.cos(angle),
      y: center + radius * Math.sin(angle),
    })
  }

  const cliqueSet = plantedClique.value ? new Set(plantedClique.value) : null
  const cliqueEdges: [number, number][] = []
  if (cliqueSet && plantedClique.value) {
    const c = plantedClique.value
    for (let i = 0; i < c.length; i++) {
      for (let j = i + 1; j < c.length; j++) {
        cliqueEdges.push([c[i], c[j]])
      }
    }
  }

  const baseSet = new Set(baseEdges.value.map(([a, b]) => `${a},${b}`))
  const blend = cliqueSet ? highlightBlend.value : 0

  ctx.clearRect(0, 0, size, size)

  // ER の辺を描画（薄く）
  ctx.strokeStyle = 'rgba(25, 118, 210, 0.35)'
  ctx.lineWidth = 0.8
  for (const [i, j] of baseEdges.value) {
    const a = positions[i]
    const b = positions[j]
    ctx.beginPath()
    ctx.moveTo(a.x, a.y)
    ctx.lineTo(b.x, b.y)
    ctx.stroke()
  }

  // 埋め込みクリークの辺を描画（blend で赤→青を補間）
  for (const [i, j] of cliqueEdges) {
    if (baseSet.has(`${i},${j}`)) continue
    const a = positions[i]
    const b = positions[j]
    const r = Math.round(211 * blend + 25 * (1 - blend))
    const g = Math.round(47 * blend + 118 * (1 - blend))
    const b_ = Math.round(47 * blend + 210 * (1 - blend))
    const a_ = 0.9 * blend + 0.35 * (1 - blend)
    ctx.strokeStyle = `rgba(${r},${g},${b_},${a_})`
    ctx.lineWidth = 1.4 * blend + 0.8 * (1 - blend)
    ctx.beginPath()
    ctx.moveTo(a.x, a.y)
    ctx.lineTo(b.x, b.y)
    ctx.stroke()
  }

  // 頂点を描画（blend でクリーク頂点を赤→青に補間）
  for (let i = 0; i < positions.length; i++) {
    const pos = positions[i]
    const isClique = cliqueSet?.has(i)
    if (isClique && blend > 0) {
      const r = Math.round(211 * blend + 25 * (1 - blend))
      const g = Math.round(47 * blend + 118 * (1 - blend))
      const b_ = Math.round(47 * blend + 210 * (1 - blend))
      ctx.fillStyle = `rgb(${r},${g},${b_})`
    } else {
      ctx.fillStyle = '#1976d2'
    }
    ctx.beginPath()
    ctx.arc(pos.x, pos.y, nodeRadius, 0, 2 * Math.PI)
    ctx.fill()
  }
}

onMounted(() => {
  initBaseEdges()
  draw()
})
watch(() => [props.n, props.p, props.seed, props.size], () => {
  if (highlightTimeout) {
    clearTimeout(highlightTimeout)
    highlightTimeout = null
  }
  stopTransition()
  initBaseEdges()
  plantedClique.value = null
  highlightClique.value = false
  highlightBlend.value = 0
  draw()
})
watch([plantedClique, highlightClique, highlightBlend], () => draw())
</script>

<style scoped>
.er-graph-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}
.er-canvas {
  border-radius: 8px;
  max-width: 100%;
  height: auto;
}
.er-buttons {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  justify-content: center;
}
.er-btn {
  padding: 0.35rem 0.75rem;
  font-size: 0.8rem;
  border-radius: 6px;
  border: 1px solid #ccc;
  background: #f5f5f5;
  cursor: pointer;
  color: #333;
}
.er-btn:hover {
  background: #eee;
}
.er-btn-pc:hover {
  background: #ffebee;
  border-color: #d32f2f;
  color: #b71c1c;
}
.er-btn-restore:hover {
  background: #e3f2fd;
  border-color: #1976d2;
  color: #0d47a1;
}
.er-caption {
  font-size: 0.85rem;
  color: var(--slidev-theme-primary, #1976d2);
  margin: 0;
}
</style>
