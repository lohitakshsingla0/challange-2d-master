<script setup>
import { onMounted } from 'vue'

let canvasElement = null
let canvas = null
let origin = { x: 250, y: 250 }
let panAction = false
let moveObjectAction = false
let lastActionPostion = null
let scale = 1.0   //used for pi to mm
let circles = []

onMounted(() => {
  canvasElement = document.getElementById('myCanvas2D')
  canvas = canvasElement.getContext('2d')
  repaint()
})

function repaint() {
  // clear the screen and set the background color
  canvas.fillStyle = 'lightgray'
  canvas.fillRect(0, 0, canvasElement.width, canvasElement.height)

  // mark the origin
  canvas.beginPath()
  canvas.strokeStyle = 'black'
  canvas.moveTo(origin.x - 10, origin.y)
  canvas.lineTo(origin.x + 10, origin.y)
  canvas.moveTo(origin.x, origin.y - 10)
  canvas.lineTo(origin.x, origin.y + 10)
  canvas.stroke()

  // paint the circles

  for (const circle of circles) {
    canvas.beginPath()
    canvas.strokeStyle = 'red'
    if (circle.highlight) canvas.strokeStyle = 'blue'
    canvas.arc(circle.x*scale + origin.x, circle.y*scale + origin.y, circle.d*scale, 0, 2 * Math.PI)
    canvas.stroke()
  }
}

function getMousePos(evt) {
  var rect = canvasElement.getBoundingClientRect()
  return { x: evt.clientX - rect.left, y: evt.clientY - rect.top }
}

function doubleClick(event) {
  const pos = getMousePos(event)
  // TODO scale from pixel to mm
  circles.push({ x: pos.x - origin.x*scale, y: pos.y - origin.y*scale, d: 50 })
  repaint()
}

function calcDistance(a, b) {
  return Math.sqrt(Math.pow(b.x - a.x, 2) + Math.pow(b.y - a.y, 2))
}

function mouseMove(event) {
  const mousePos = getMousePos(event)

  let diff = { x: 0, y: 0 }

  if (panAction || moveObjectAction) {
    if (lastActionPostion != null) {
      diff = { x: mousePos.x - lastActionPostion.x, y: mousePos.y - lastActionPostion.y }
    }
    lastActionPostion = mousePos
  }

  if (panAction) {
    origin = { x: origin.x + diff.x, y: origin.y + diff.y }
  } else if (moveObjectAction) {
    for (let circle of circles) {
      // move highlighted circle
      if (circle.highlight) {
        circle.x = circle.x + diff.x
        circle.y = circle.y + diff.y
        break
      }
    }
  } else {
    // highlight the closest circle
    let currentMinDistance = Number.MAX_VALUE
    let currentHighlightCircle = null
    for (const circle of circles) {
      circle.highlight = undefined
      const circlePos = { x: circle.x + origin.x, y: circle.y + origin.y }
      const distance = calcDistance(circlePos, mousePos)
      if (distance <= circle.d) {
        if (distance < currentMinDistance) {
          currentHighlightCircle = circle
          currentMinDistance = distance
        }
      }
    }
    if (currentHighlightCircle != null) currentHighlightCircle.highlight = true
  }
  repaint()
}

function mouseDown(event) {
  if (event.button == 2) panAction = true
  if (event.button == 0) moveObjectAction = true
}
function mouseUp(event) {
  panAction = false
  moveObjectAction = false
  lastActionPostion = null
}

function clear() {
  circles = []
  repaint()
}

function zoomIn() {
  scale = scale*1.2
  repaint()
}
function zoomOut() {
  scale = scale * 0.8
  repaint()
 }







</script>

<template>
  <canvas
    width="500"
    height="500"
    id="myCanvas2D"
    @mousemove="mouseMove"
    @dblclick.prevent="doubleClick"
    @contextmenu.prevent="rightClick"
    @mousedown="mouseDown"
    @mouseup="mouseUp"
    @mouseleave="mouseLeave"
    @mouseenter="mouseEnter"
  ></canvas>
  <div class="text-sm">left mouse - move circle / right mouse - pan screen / double click - add circle</div>
  <div><button @click="clear">clear</button><button @click="zoomIn">+</button><button @click="zoomOut">-</button></div>
</template>
