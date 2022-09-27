<script setup>
import { PerspectiveCamera, Scene, WebGLRenderer } from 'three'
import * as THREE from 'three'
import WebGL from 'three/examples/jsm/capabilities/WebGL'
import { gsap } from 'gsap'
// import { ScrollSmoother } from 'gsap/ScrollSmoother'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

const canvas = ref(null)
const messageContainer = ref(null)

let scene; let camera; let renderer; let box; let model; let controls; let mixer; let thorus; let stats; const objects = []

const getMouseDegrees = (x, y, degreeLimit) => {
  let dx = 0
  let dy = 0
  let xdiff
  let xPercentage
  let ydiff
  let yPercentage

  const w = { x: window.innerWidth, y: window.innerHeight }

  // Left (Rotates neck left between 0 and -degreeLimit)

  // 1. If cursor is in the left half of screen
  if (x <= w.x / 2) {
    // 2. Get the difference between middle of screen and cursor position
    xdiff = w.x / 2 - x
    // 3. Find the percentage of that difference (percentage toward edge of screen)
    xPercentage = (xdiff / (w.x / 2)) * 100
    // 4. Convert that to a percentage of the maximum rotation we allow for the neck
    dx = ((degreeLimit * xPercentage) / 100) * -1
  }
  // Right (Rotates neck right between 0 and degreeLimit)
  if (x >= w.x / 2) {
    xdiff = x - w.x / 2
    xPercentage = (xdiff / (w.x / 2)) * 100
    dx = (degreeLimit * xPercentage) / 100
  }
  // Up (Rotates neck up between 0 and -degreeLimit)
  if (y <= w.y / 2) {
    ydiff = w.y / 2 - y
    yPercentage = (ydiff / (w.y / 2)) * 100
    // Note that I cut degreeLimit in half when she looks up
    dy = (((degreeLimit * 0.5) * yPercentage) / 100) * -1
  }

  // Down (Rotates neck down between 0 and degreeLimit)
  if (y >= w.y / 2) {
    ydiff = y - w.y / 2
    yPercentage = (ydiff / (w.y / 2)) * 100
    dy = (degreeLimit * yPercentage) / 100
  }
  return { x: dx, y: dy }
}

const onMouseMove = (event) => {
  const degrees = getMouseDegrees(event.clientX, event.clientY, 50)
  if (thorus) {
    thorus.rotation.y = THREE.MathUtils.degToRad(degrees.x)
    thorus.rotation.x = THREE.MathUtils.degToRad(degrees.y)
    // thorus.rotation.x = (THREE.MathUtils.degToRad(degrees.y) - thorus.rotation.y) * 0.005
    // thorus.rotation.y = (THREE.MathUtils.degToRad(degrees.x) - thorus.rotation.x) * 0.005
  }
}
const onMouseEnd = (event) => {
//   mouse = null
}

const init = (canvas) => {
  const CANVAS_WIDTH = window.innerWidth
  const CANVAS_HEIGHT = window.innerHeight

  // Renderer setup
  renderer = new WebGLRenderer({ antialias: true, canvas })

  // Scene setup
  scene = new Scene()
  scene.background = new THREE.Color(0x000000)

  // Camera setup
  camera = new PerspectiveCamera(60, CANVAS_WIDTH / CANVAS_HEIGHT, 0.1, 300)
  camera.position.z = 18

  // Create thorus knot geometry
  const radius = 3.5 // ui: radius
  const tubeRadius = 1.5 // ui: tubeRadius
  const radialSegments = 8 // ui: radialSegments
  const tubularSegments = 64 // ui: tubularSegments
  const p = 2 // ui: p
  const q = 3 // ui: q
  const geometry = new THREE.TorusKnotGeometry(
    radius, tubeRadius, tubularSegments, radialSegments, p, q)

  //   const radius = 7 // ui: radius
  //   const widthSegments = 36 // ui: widthSegments
  //   const heightSegments = 30 // ui: heightSegments
  //   const geometry = new THREE.SphereGeometry(radius, widthSegments, heightSegments)
  thorus = new THREE.Mesh(geometry, new THREE.MeshStandardMaterial({
    color: 0xFFFFFF,
    roughness: 0.6,
    metalness: 0.1,
  }))
  thorus.position.set(0, 0, 0)
  scene.add(thorus)

  {
    const light = new THREE.DirectionalLight(0x6328BD, 0.5)
    light.position.set(0, 20, 0)
    light.up.set(0, 0, 1)
    light.lookAt(0, 0, 0)
    scene.add(light)
  }
  //   const environment = new RoomEnvironment()
  //   const pmremGenerator = new THREE.PMREMGenerator(renderer)

  //   scene.environment = pmremGenerator.fromScene(environment).texture

  // Add mouse move event to canvas
  canvas.addEventListener('mousemove', onMouseMove, false)
  canvas.addEventListener('mouseenter', onMouseMove, false)
  canvas.addEventListener('dragover', onMouseMove, false)
  canvas.addEventListener('mouseup', onMouseEnd, false)
  canvas.addEventListener('mouseleave', onMouseEnd, false)
}

// Resize check function
const resizeRendererToDisplaySize = (renderer) => {
  const canvas = renderer.domElement
  const width = canvas.clientWidth
  const height = canvas.clientHeight
  const needResize = canvas.width !== width || canvas.height !== height
  if (needResize)
    renderer.setSize(width, height, false)

  return needResize
}

// Render scene
const animate = (time) => {
  // Responsive fix
  if (resizeRendererToDisplaySize(renderer)) {
    const canvas = renderer.domElement
    camera.aspect = canvas.clientWidth / canvas.clientHeight
    camera.updateProjectionMatrix()
  }

  thorus.position.z = Math.cos(time / 800) * 0.5

  renderer.render(scene, camera)
}

// GS Reveal Hero text
const hide = (elem) => {
  gsap.set(elem, { autoAlpha: 0 })
}
const animateFrom = (elem) => {
  // Get delay value from data attribute
  const delay = elem.dataset.delay || 0
  const duration = elem.dataset.duration || 1.25
  const x = elem.dataset.x || 0
  const y = elem.dataset.y || 0
  const scale = elem.dataset.scale || 1
  const ease = elem.dataset.ease || 'power3.out'

  gsap.fromTo(elem, { x, y, scale, autoAlpha: 0 }, {
    duration,
    delay,
    ease,
    x: 0,
    y: 0,
    scale: 1,
    autoAlpha: 1,
    overwrite: 'auto',
  })
}
const initGSReveal = () => {
  const elems = document.querySelectorAll('.gs_reveal')
  elems.forEach((elem) => {
    hide(elem)

    ScrollTrigger.create({
      trigger: elem,
      onEnter() { animateFrom(elem) },
      once: true,
    })
  })
}

const initGSParallax = () => {
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: '#hero',
      start: 'top top',
      end: 'bottom top',
      scrub: true,
    },
  })

  gsap.utils.toArray('.parallax').forEach((layer) => {
    const depth = layer.dataset.depth
    const movement = -(layer.offsetHeight * depth)
    tl.to(layer, { y: movement, ease: 'none' }, 0)
  })
}

onBeforeMount(() => {
  initGSReveal()
  initGSParallax()
})
onMounted(() => {
  if (WebGL.isWebGLAvailable()) {
    init(canvas.value)
    renderer.setAnimationLoop(animate)
  }
  else {
    const warning = WebGL.getWebGLErrorMessage()
    messageContainer.value.appendChild(warning)
  }
})
</script>

<template>
  <div id="hero" class="header h-screen w-full bg-black text-white relative">
    <div class="canvas-container absolute top-0 left-0 w-full h-full z-10 gs_reveal parallax" :data-scale="0.4" :data-duration="3" data-depth="0.80">
      <canvas ref="canvas" class="w-full h-full block" />
      <div ref="messageContainer" class="relative" />
    </div>
    <div class="absolute top-0 left-0 w-full h-full z-20 pointer-events-none parallax" data-depth="0.10">
      <div class="c-container mx-auto w-full py-16 pt-24 md:py-24 flex flex-col justify-between h-full">
        <div class="w-full flex justify-between items-center">
          <div class="absolute top-8 left-1/2 transform -translate-x-1/2 md:relative md:top-auto md:translate-x-0 md:left-auto">
            <nuxt-link to="/" class="pointer-events-auto text-3xl md:text-4xl font-semibold gs_reveal" data-y="100%">
              <span>Ilaryux.</span>
            </nuxt-link>
          </div>

          <app-button class="pointer-events-auto" tag="nuxt-link" to="/">
            Let's Talk
          </app-button>
        </div>
        <div>
          <h1 class="text-6xl md:text-7xl font-semibold mb-8 md:mb-12 overflow-hidden leading-tight">
            <span class="gs_reveal inline-block relative" data-y="100%" :data-duration="2">
              Shaping&nbsp;
            </span>
            <br class="md:hidden">
            <span class="gs_reveal inline-block relative" data-y="100%" :data-delay="0.2" :data-duration="2">
              The&nbsp;
            </span>
            <span class="gs_reveal inline-block relative" data-y="100%" :data-delay="0.4" :data-duration="2">
              Brands
            </span>
          </h1>
          <p class="text-lg md:text-xl max-w-md gs_reveal" data-y="100%">
            Providing creative digital experience
            to help organizations innovate their business.
          </p>
        </div>

        <div class="w-full flex justify-between">
          <app-button class="pointer-events-auto" tag="nuxt-link" to="/">
            Featured
          </app-button>

          <app-button class="pointer-events-auto" tag="nuxt-link" to="/">
            Services
          </app-button>
        </div>
      </div>
    </div>
  </div>
</template>
