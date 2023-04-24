<script setup lang="ts">
import {
  Scene,
  PerspectiveCamera,
  Mesh,
  SphereGeometry,
  MeshBasicMaterial,
  WebGLRenderer,
  Color,
  Fog,
  AmbientLight,
} from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import gsap from 'gsap'
import { useWindowSize } from '@vueuse/core'

const { width, height } = useWindowSize()
const aspectRatio = computed(() => width.value / height.value)
// create canvas ref
const experienceRef: Ref<HTMLCanvasElement | null> = ref(null)

const bgColor = new Color('#E1F0C2')

// init scene
const scene = new Scene()
scene.fog = new Fog(bgColor, 0.1, 75)
scene.background = bgColor

// init renderer
let renderer: WebGLRenderer
let controls: OrbitControls

// init camera
// const width = 800
// const height = 600
const camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000)
camera.position.set(0, 0, 3)

//add camera to scene
scene.add(camera)

//add light to scene
//const light = new DirectionalLight(0xffffff, 1)
const ambientLight = new AmbientLight(0xffffff, 0.5)
scene.add(ambientLight)

// load GLTF model from composable
const { load } = useGLTFModel()
const { scene: model } = await load('/nuxty/nuxty.gltf')
scene.add(model)

// create bjects
const sphere = new Mesh(
  new SphereGeometry(1, 32, 32),
  new MeshBasicMaterial({ color: 0x008080 })
)
// add objects to scene
scene.add(sphere)

function updateCamera() {
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
}

function updateRenderer() {
  renderer.setSize(width.value, height.value)
  renderer.render(scene, camera)
}

function setRenderer() {
  if (experienceRef.value) {
    // create renderer
    renderer = new WebGLRenderer({
      canvas: experienceRef.value,
      alpha: true,
      antialias: true,
    })

    // antialiasing
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

    // set controls
    controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    updateRenderer()
  }
}
const loop = () => {
  //sphere.position.x += 0.001
  controls.update()
  renderer.render(scene, camera)
  window.requestAnimationFrame(loop)
}

onMounted(async () => {
  await nextTick()

  setRenderer()
  loop()
  gsap.to(sphere.position, { x: 2, duration: 4 })
  gsap.to(sphere.scale, { x: 0.5, y: 0.5, z: 0.5, duration: 4 })
  gsap.to(model.position, { x: 1, duration: 4 })
})

watch(aspectRatio, () => {
  updateCamera()
  updateRenderer()
})
</script>

<template>
  <div class="exp-holder">
    <canvas ref="experienceRef" />
  </div>
</template>

<style lang="scss">
.exp-holder {
}
</style>
