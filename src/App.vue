<script setup lang="ts">
// import { isDark } from './composables'

const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)

const WIDTH = 400
const HEIGHT = 400

interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  theta: number
}

function init() {
  // let color = '#fff'
  // if (!isDark.value) color = 'gray'
  ctx.strokeStyle = 'gray'

  step({
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 30,
    theta: -Math.PI / 2,
  })
}

const pendingTasks: Function[] = []

function step(b: Branch, depth = 0) {
  const end = getEndPoint(b)
  drawBranch(b)

  if (depth < 4 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta - 0.4 * Math.random(),
    }, depth + 1))
  }
  if (depth < 4 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta + 0.4 * Math.random(),
    }, depth + 1))
  }
}

function frame() {
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}

let frameCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    frameCount += 1
    if (frameCount % 3 === 0)
      frame()
    startFrame()
  })
}

startFrame()

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
  const end = getEndPoint(b)
  lineTo(b.start, end)
}

onMounted(() => {
  init()
  setTimeout(() => {
    location.reload()
  }, 5000)
})
</script>

<template>
  <main
    font-sans p="x-4 y-10" text="center gray-700 dark:gray-200"
  >
    <div
      mt-80px
      flex="~"
      items-center justify-center
    >
      <canvas ref="el" width="400" height="400" />
    </div>
    <Footer />
  </main>
</template>
