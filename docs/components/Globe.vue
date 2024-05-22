<script setup lang="ts">
import { useEventListener } from '@vueuse/core'
import ThreeGlobe from 'three-globe'
import { PerspectiveCamera, DirectionalLight, AmbientLight, WebGLRenderer, Scene, Color, Fog } from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { CSS2DRenderer } from 'three/addons/renderers/CSS2DRenderer.js'

// maybe in the future, we could use globeImageUrl instead of hexPolygonsData for performance optimization
import countries from './files/globe-data.json'

// the number of arcs to display on the globe
const arcs = 13
const element = ref()
const colorMode = useColorMode()
let renderers, scene, camera, controls

let Globe: ThreeGlobe

const globeSize = [
  {
    breakpoint: 1024,
    size: 550,
  },
  {
    breakpoint: 768,
    size: 500,
  },
  {
    breakpoint: 640,
    size: 200,
  },
]
const arcsData = [...Array(arcs).keys()].map(() => ({
  startLat: (Math.random() - 0.5) * 180,
  startLng: (Math.random() - 0.5) * 360,
  endLat: (Math.random() - 0.5) * 180,
  endLng: (Math.random() - 0.5) * 360,
}))

onMounted(() => init())

function init() {
  renderers = [new WebGLRenderer({ antialias: true }), new CSS2DRenderer()]
  renderers.forEach((r, idx) => {
    r.setSize(globeSize[0].size, globeSize[0].size)
    if (idx === 0) {
      r.setPixelRatio(window.devicePixelRatio)
      // set BG transparent
      r.setClearColor(0x000000, 0)
    } else {
      // overlay additional on top of main renderer
      r.domElement.style.position = 'absolute'
      r.domElement.style.top = '0px'
      r.domElement.style.pointerEvents = 'none'
    }
    element.value.appendChild(r.domElement)
  })

  // Setup scene
  scene = new Scene()
  scene.add(new AmbientLight(0xBBBBBB, 0.3))

  // Setup camera
  camera = new PerspectiveCamera()
  camera.updateProjectionMatrix()

  // lights
  const dLight = new DirectionalLight(0xFFFFFF, 0.8)
  dLight.position.set(-800, 2000, 400)
  camera.add(dLight)

  const dLight1 = new DirectionalLight(0x7982F6, 1)
  dLight1.position.set(-200, 500, 200)
  camera.add(dLight1)

  camera.position.z = 300
  camera.position.x = 0
  camera.position.y = 0

  scene.add(camera)

  // Effects
  scene.fog = new Fog(0x535EF3, 400, 2000)

  controls = new OrbitControls(camera, renderers[0].domElement)
  controls.enableDamping = true
  controls.dynamicDampingFactor = 0.01
  controls.enablePan = false
  controls.rotateSpeed = 0.8
  controls.enableZoom = false
  controls.autoRotate = true
  controls.autoRotateSpeed = -2

  // Limit camera angles
  controls.minPolarAngle = Math.PI / 3.5
  controls.maxPolarAngle = Math.PI - Math.PI / 3


  initGlobe()
  useEventListener(window, 'resize', onWindowResize)
  animate()
}

function initGlobe() {
  Globe = new ThreeGlobe({
    waitForGlobeReady: true,
    animateIn: true,
  })
    // surface
    .hexPolygonsData(countries.features)
    .hexPolygonResolution(3)
    .hexPolygonMargin(0.7)
    .showAtmosphere(true)
    .atmosphereColor('#3a228a')
    .atmosphereAltitude(0.25)

  setTimeout(() => {
    Globe
      // arcs
      .arcsData(arcsData)
      .arcColor(() => '#00DC82')
      .arcStroke(0.3)
      .arcDashLength(0.9)
      .arcDashGap(4)
      .arcDashInitialGap(() => Math.random() * 5)
      .arcDashAnimateTime(1000)
      .arcsTransitionDuration(1000)
      // points
      .pointsData(arcsData.flatMap(arc => [
        { lat: arc.startLat, lng: arc.startLng, },
        { lat: arc.endLat, lng: arc.endLng, },
      ]))
      .pointColor(() => '#F89D35')
      .pointAltitude(0.001)
      .pointRadius(1)
      // markers for parsing html elements
      // .htmlElementsData(arcsData.flatMap(arc => [
      //   { lat: arc.startLat, lng: arc.startLng, isOdd: 0 },
      //   { lat: arc.endLat, lng: arc.endLng, isOdd: 1 },
      // ]))
      // .htmlElement((item) => {
      //   const el = document.createElement('div')
      //   el.innerHTML = item.isOdd ? marker : cloudflare
      //   el.style.color = item.isOdd ? '#00DB86' : '#f38020'
      //   el.style.width = '20px'
      //   return el
      // })
  }, 100)

  const globeMaterial = Globe.globeMaterial()

  // change globe properties based on color mode
  watch(colorMode, () => {
    if (colorMode.value === 'dark') {
      globeMaterial.color = new Color(0x121A33)
      Globe.hexPolygonColor(() => 'rgba(255,255,255, 0.7)')
    } else {
      globeMaterial.color = new Color(0xffffff)
      Globe.hexPolygonColor(() => 'rgba(0,0,0, 0.7)')
    }
  }, { immediate: true })

  globeMaterial.emissive = new Color(0x220038)
  globeMaterial.emissiveIntensity = 0.1
  globeMaterial.shininess = 0.7

  scene.add(Globe)

  // Update pov when camera moves
  // @ts-ignore
  Globe.setPointOfView(camera.position, Globe.position)
  // @ts-ignore
  controls.addEventListener('change', () => Globe.setPointOfView(camera.position, Globe.position))
}

function onWindowResize() {
  // TODO: update globe size based on screen size
}

function animate() {
  controls.update()
  renderers.forEach((r: any) => r.render(scene, camera))
  requestAnimationFrame(animate)
}
</script>

<template>
  <div ref="element" class="absolute right-4 top-52 z-10" />
</template>
