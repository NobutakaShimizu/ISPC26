<template>
  <div class="er-slider-wrap">
    <canvas ref="canvas" :width="size" :height="size" class="er-canvas"></canvas>
    <div class="er-slider-row">
      <span class="er-slider-label">k = {{ k }}</span>
      <input
        v-model.number="sliderValue"
        type="range"
        min="0"
        max="100"
        class="er-slider"
        @input="draw"
      />
      <span class="er-slider-hint">左: k=0　右: k=100</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed } from 'vue'

const props = withDefaults(
  defineProps<{
    n?: number
    p?: number
    seed?: number
    size?: number
  }>(),
  {
    n: 100,
    p: 0.06,
    seed: 42,
    size: 300,
  }
)

// つまみ左 = 0 → k=0、つまみ右 = 100 → k=100
const sliderValue = ref(0)
const k = computed(() => sliderValue.value)

const canvas = ref<HTMLCanvasElement | null>(null)
const baseEdges = ref<[number, number][]>([])
// クリークに含める頂点の順序（固定。k を増やすと先頭から順に追加される）
const cliqueOrder = ref<number[]>([])

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

// シード付きで頂点の順列を1つ生成（k を変えても同じ順序でクリークが伸びる）
function buildCliqueOrder(n: number, seed: number): number[] {
  const rng = mulberry32(seed + 12345)
  const arr = Array.from({ length: n }, (_, i) => i)
  for (let i = 0; i < n; i++) {
    const j = i + Math.floor(rng() * (n - i))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

function initBaseEdges() {
  baseEdges.value = buildERGraph(props.n, props.p, props.seed)
  cliqueOrder.value = buildCliqueOrder(props.n, props.seed)
}

function draw() {
  if (!canvas.value) return
  const ctx = canvas.value.getContext('2d')
  if (!ctx) return

  const { n, size } = props
  const currentK = k.value
  const order = cliqueOrder.value
  const cliqueVertices = currentK <= 0 ? [] : order.slice(0, currentK)
  const cliqueSet = new Set(cliqueVertices)

  const cliqueEdges: [number, number][] = []
  for (let i = 0; i < cliqueVertices.length; i++) {
    for (let j = i + 1; j < cliqueVertices.length; j++) {
      cliqueEdges.push([cliqueVertices[i], cliqueVertices[j]])
    }
  }

  const baseSet = new Set(baseEdges.value.map(([a, b]) => `${a},${b}`))

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

  ctx.clearRect(0, 0, size, size)

  // ER の辺を描画
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

  // 埋め込みクリークの辺（ER に無いものだけ追加）
  for (const [i, j] of cliqueEdges) {
    if (baseSet.has(`${i},${j}`)) continue
    const a = positions[i]
    const b = positions[j]
    ctx.strokeStyle = 'rgba(25, 118, 210, 0.35)'
    ctx.lineWidth = 0.8
    ctx.beginPath()
    ctx.moveTo(a.x, a.y)
    ctx.lineTo(b.x, b.y)
    ctx.stroke()
  }

  // 頂点（すべて同じ色）
  ctx.fillStyle = '#1976d2'
  for (const pos of positions) {
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
  initBaseEdges()
  draw()
})
watch(k, () => draw())
</script>

<style scoped>
.er-slider-wrap {
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
.er-slider-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
  justify-content: center;
  width: 100%;
  max-width: 520px;
}
.er-slider-label {
  font-size: 0.9rem;
  font-weight: 500;
  min-width: 3rem;
  color: var(--slidev-theme-primary, #1976d2);
}
.er-slider {
  flex: 1;
  min-width: 280px;
  accent-color: var(--slidev-theme-primary, #1976d2);
}
.er-slider-hint {
  font-size: 0.7rem;
  color: #666;
  white-space: nowrap;
}
</style>
