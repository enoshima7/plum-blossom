
<script setup lang="ts">
interface Point {
  x: number
  y: number
}
interface Branch {
  start: Point
  length: number
  theta: number
}
const WIDTH = 400
const HEIGHT = 400
const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)
const startPoint: Point = { x: WIDTH / 2, y: HEIGHT }
const pendingTasks: Function[] = []
function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}

function drawBranch(b: Branch) {
  const { start } = b
  const end = getBranchEndPoint(b)
  lineTo(start, end)
}
function getBranchEndPoint(b: Branch) {
  const { start, length, theta } = b
  const end = {
    x: start.x + length * Math.cos(theta),
    y: start.y + length * Math.sin(theta),
  }
  return end
}

function init() {
  ctx.strokeStyle = '#fff'
  const branch: Branch = {
    start: startPoint,
    length: 10,
    theta: -Math.PI / 2,
  }
  step(branch)
}

function step(b: Branch, depth = 0) {
  drawBranch(b)
  const endPoint = getBranchEndPoint(b)
  if (depth < 5 || Math.random() < 0.3) {
    pendingTasks.push(() => {
      const leftBranch: Branch = {
        start: endPoint,
        length: b.length + (Math.random() * 20 - 10),
        theta: b.theta - 0.4 * Math.random(),
      }
      return step(leftBranch, depth + 1)
    })
  }
  if (depth < 5 || Math.random() < 0.3) {
    pendingTasks.push(() => {
      const rightBranch = {
        start: endPoint,
        length: b.length + (Math.random() * 10 - 5),
        theta: b.theta + 0.3 * Math.random(),
      }
      return step(rightBranch, depth + 1)
    })
  }
}
let frameCount = 0
function frame() {
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}
function startFrame() {
  requestAnimationFrame(() => {
    frameCount++
    if (frameCount % 3 === 0)
      frame()

    startFrame()
  })
}
startFrame()
onMounted(() => {
  init()
})
</script>

<template>
  <canvas ref="el" :width="WIDTH" :height="HEIGHT" border border-white />
</template>

<style scoped>
</style>
