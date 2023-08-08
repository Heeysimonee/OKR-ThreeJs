<script setup lang="ts">
import {
  BoxGeometry,
  Mesh,
  MeshBasicMaterial,
  Scene,
  WebGLRenderer,
  PerspectiveCamera
} from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { useWindowSize } from '@vueuse/core'
import { computed, onMounted, ref, watch } from 'vue'

const { width, height } = useWindowSize()
const myCanvas = ref<null | HTMLCanvasElement>()

//Set up what and where is to be rendered by three.js
const scene = new Scene()
const aspectRatio = computed(() => width.value / height.value)
let renderer: WebGLRenderer

let camera: PerspectiveCamera

//A mesh is always composed of a geometry and a material
const square = new Mesh(new BoxGeometry(1, 1, 1), new MeshBasicMaterial({ color: '#00ff00' }))
scene.add(square)

//This projection mode is designed to mimic the way the human eye sees. 
//It is the most common projection mode used for rendering a 3D scene.
camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000)
camera.position.set(0, 0, 5)
scene.add(camera)
function updateRenderer() {
  renderer.setSize(width.value, height.value)
  renderer.setPixelRatio(window.devicePixelRatio)
}

function updateCamera() {
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
}

//resize the scene based on screen size
watch(aspectRatio, updateRenderer)
watch(aspectRatio, updateCamera)

//render the scene based on screen FPS
const loop = () => {
  square.rotation.y += 0.001
  square.rotation.x += 0.01

  renderer.render(scene, camera)
  requestAnimationFrame(loop)
}

onMounted(() => {
  //The WebGL renderer displays your beautifully crafted scenes using WebGL
  renderer = new WebGLRenderer({
    canvas: myCanvas.value as HTMLCanvasElement,
    antialias: true
  })
  updateRenderer()
  updateCamera()
  loop()
})
</script>

<template>
  <canvas ref="myCanvas"></canvas>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

html,
body {
  height: 100vh;
  font-family: 'Poppins';
  background: rgb(24, 24, 24);
}
body {
  overflow-x: hidden;
}
canvas {
  position: absolute;
  top: 0;
  left: 0;
  outline: none;
}
</style>
