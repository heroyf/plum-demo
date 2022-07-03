<script setup lang='ts'>
import { useRafFn } from '@vueuse/core'
const init = ref<number>(5)
const len = ref<number>(10)
const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)
const stopped = ref(false)
const MAX_LEN = 600
let depth = 0
let pendingTasks: Function[] = []
const framesCount = 0
const controls = useRafFn(() => {
  if (framesCount % 3 === 0)
    frame()
}, { immediate: false })
interface Branch {
  start: Point
  length: number
  theta: number
}
interface Point {
  x: number
  y: number
}
const f = {
  start: () => {},
}
watch([init, len], () => f.start())

function frame() {
  depth += 1
  const tasks = [...pendingTasks]
  pendingTasks = []
  if (!tasks.length) {
    controls.pause()
    stopped.value = true
  }
  tasks.forEach(fn => fn())
}

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}

function getEndPoint(b: Branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.theta),
    y: b.start.y + b.length * Math.sin(b.theta),
  }
}

function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}
onMounted(async () => {
  const step = (b: Branch) => {
    const end = getEndPoint(b)
    drawBranch(b)
    // 小于init的时候，100%会生成新的分支
    if (depth < init.value || Math.random() < 0.5) {
      pendingTasks.push(() => step({
        start: end,
        length: b.length,
        theta: b.theta - Math.PI / 12,
      }))
    }
    if (depth < init.value || Math.random() < 0.5) {
      pendingTasks.push(() => step({
        start: end,
        length: b.length,
        theta: b.theta + Math.PI / 12,
      }))
    }
  }
  f.start = () => {
    controls.pause()
    depth = 0
    ctx.strokeStyle = '#fff'
    ctx.lineWidth = 1
    ctx.clearRect(0, 0, MAX_LEN, MAX_LEN)
    pendingTasks = Math.random() < 0.5
      ? [
          () => step({ start: { x: 0, y: Math.random() * MAX_LEN }, length: Math.random() * len.value, theta: 0 }),
          () => step({ start: { x: MAX_LEN, y: Math.random() * MAX_LEN }, length: Math.random() * len.value, theta: Math.PI }),
        ]
      : [
          () => step({ start: { x: Math.random() * MAX_LEN, y: 0 }, length: 10, theta: Math.PI / 2 }),
          () => step({ start: { x: Math.random() * MAX_LEN, y: MAX_LEN }, length: 10, theta: -Math.PI / 2 }),
        ]
    controls.resume()
    stopped.value = false
  }
  f.start()
})
</script>

<template>
  <canvas ref="el" width="600" height="600" border @click="f.start" />
  <div flex>
    <div flex>
      <div text-gray-400>
        init
      </div>
      <div text-gray-500 bold px-2 cursor-pointer @click="init = init % 8 + 1">
        {{ init }}
      </div>
    </div>
    <div flex>
      <div text-gray-400>
        len
      </div>
      <div text-gray-500 bold px-2 cursor-pointer @click="len = len % 10 + 1">
        {{ len }}
      </div>
    </div>
  </div>
</template>
