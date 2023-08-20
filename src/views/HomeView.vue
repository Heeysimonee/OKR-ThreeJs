<script setup lang="ts">
import {
  BoxGeometry,
  Mesh,
  MeshBasicMaterial,
  Scene,
  WebGLRenderer,
  PerspectiveCamera,
  MeshNormalMaterial,
  AxesHelper,
  Quaternion,
  Group,
  Vector3,
  Clock
} from 'three'
import { useWindowSize } from '@vueuse/core'
import { computed, onMounted, ref, watch } from 'vue'
import { MathUtils } from 'three/src/math/MathUtils.js';
import gsap from 'gsap'

const { width, height } = useWindowSize()
const myCanvas = ref<null | HTMLCanvasElement>()

//Set up what and where is to be rendered by three.js
const scene = new Scene()
const aspectRatio = computed(() => width.value / height.value)
let renderer: WebGLRenderer

let camera: PerspectiveCamera

//A mesh is always composed of a geometry and a material
const square = new Mesh(new BoxGeometry(1, 1, 1), new MeshNormalMaterial())
scene.add(square)
const square2 = square.clone()
const square3 = square.clone()

// scene.add(square2, square3)
square2.position.x = -2
square3.position.x = 3

square2.scale.multiplyScalar(1.2)
square.scale.y = 2
square3.scale.set(0.5, 0.75, 3)

const axeshelper = new AxesHelper(2)
square.add(axeshelper.clone())
square2.add(axeshelper.clone())
square3.add(axeshelper.clone())

square2.rotation.y = MathUtils.degToRad(45) // Math.PI * 0.25
square2.rotation.z = Math.PI * 0.1
square.rotation.order = 'ZXY'

square.quaternion.copy(new Quaternion().random())

const group = new Group

group.add(square2, square3)
scene.add(group)
group.position.set(0, 1, -1)

const v = new Vector3()

square2.getWorldPosition(v)
//This projection mode is designed to mimic the way the human eye sees. 
//It is the most common projection mode used for rendering a 3D scene.
camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000)
camera.position.set(-2, 1, 5)
camera.lookAt(v)

scene.add(camera)
const velocity = 1
// const time = ref(Date.now())
const time = ref(new Clock)
square.scale.multiplyScalar(0)

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
  const deltaTime = time.value.getDelta()
  const elapsedTime = time.value.getElapsedTime()

  // square.rotation.x += velocity * deltaTime
  // square.rotation.y += velocity * deltaTime
  square.position.y = Math.sin(elapsedTime ) + 1

  camera.position.y = Math.sin(elapsedTime) 
  camera.lookAt(square.position)


  renderer.render(scene, camera)
  requestAnimationFrame(loop)
}
function pop() {
  gsap.to(square.scale, { duration: 1, x: 1, y: 1, z: 1 })
  gsap.to(square.rotation, { duration: 1, x: 3.14, y: 3.14 })
}
onMounted(() => {
  pop()
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
