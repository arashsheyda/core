<script setup lang="ts">
import { useEventListener } from '@vueuse/core'
import ThreeGlobe from 'three-globe'
import { PerspectiveCamera, DirectionalLight, AmbientLight, WebGLRenderer, Scene, Color, MeshBasicMaterial, TextureLoader, Mesh, SphereGeometry } from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

import countries from './files/globe-data.json'
import marker from './marker.svg'

// the number of arcs to display on the globe
const arcs = 13

const element = ref()
const colorMode = useColorMode()

let renderers
let scene: Scene
let Globe: ThreeGlobe
let controls: OrbitControls
let camera: PerspectiveCamera

let dLight: DirectionalLight, dLight1: DirectionalLight


const globeSize = [
  // xl
  {
    breakpoint: 1280,
    size: 900,
  },
  // lg
  {
    breakpoint: 1024,
    size: 700,
  },
  // md
  {
    breakpoint: 768,
    size: 400,
  },
  // sm
  {
    breakpoint: 640,
    size: 300,
  },
  // xs
  {
    breakpoint: 0,
    size: 250,
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
  renderers = [new WebGLRenderer({ antialias: true })]
  renderers.forEach((r, idx) => {
    // r.setSize(globeSize[0].size, globeSize[0].size)
    if (idx === 0) {
      r.setPixelRatio(window.devicePixelRatio)
      // set BG transparent
      r.setClearColor(0x000000, 0)
    } else {
      // if there are multiple renderers, do ...
    }
    element.value.appendChild(r.domElement)
  })
  onWindowResize()

  // Setup scene
  scene = new Scene()
  scene.add(new AmbientLight(0xBBBBBB, 0.3))

  // Setup camera
  camera = new PerspectiveCamera()
  camera.updateProjectionMatrix()

  // updateLights()
  watch(() => colorMode.preference, updateLights, { immediate: true })


  camera.position.z = 300
  camera.position.x = 0
  camera.position.y = 0

  scene.add(camera)

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

function updateLights() {
  scene.remove(dLight, dLight1)

  if (colorMode.preference === 'dark') {
    dLight = new DirectionalLight(0xFFFFFF, 0.8)
    dLight.position.set(-800, 2000, 400)

    dLight1 = new DirectionalLight(0x7982F6, 1)
    dLight1.position.set(-200, 500, 200)

    scene.add(dLight, dLight1)
  } else {
    //
  }
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
    // .atmosphereColor('#3a228a')
    .atmosphereAltitude(0.25)

  const coordinates = arcsData.flatMap(arc => [
    { lat: arc.startLat, lng: arc.startLng, },
    { lat: arc.endLat, lng: arc.endLng, },
  ])

  setTimeout(() => {
    Globe
      // arcs
      .arcsData(arcsData)
      .arcColor(() => '#00DB86')
      .arcStroke(0.3)
      .arcDashLength(0.9)
      .arcDashGap(4)
      .arcDashInitialGap(() => Math.random() * 5)
      .arcDashAnimateTime(1000)
      .arcsTransitionDuration(1000)
      // tile markers
      .tilesData(coordinates)
      .tileWidth(4)
      .tileHeight(4)
      .tileMaterial(() => new MeshBasicMaterial({
        map: new TextureLoader().load(marker),
        transparent: true,
        opacity: 1,
      }))
  }, 250)

  const globeMaterial = Globe.globeMaterial()

  // change globe properties based on color mode
  watch(() => colorMode.preference, () => {
    if (colorMode.preference === 'dark') {
      globeMaterial.color = new Color(0x121A33)
      Globe
        .hexPolygonColor(() => 'rgba(255,255,255, 0.7)')
        .atmosphereColor('#3a228a')

      globeMaterial.emissive = new Color(0x220038)
      globeMaterial.emissiveIntensity = 0.1
    } else {
      globeMaterial.color = new Color(0xffffff)
      Globe
        .hexPolygonColor(() => 'rgba(0,0,0, 0.7)')
        .atmosphereColor('#E6E6E6')

      globeMaterial.emissive = new Color(0xffffff)
      globeMaterial.emissiveIntensity = 0.7
    }
  }, { immediate: true })

  globeMaterial.shininess = 0.7

  scene.add(Globe)

  // Update pov when camera moves
  // @ts-ignore
  Globe.setPointOfView(camera.position, Globe.position)
  // @ts-ignore
  controls.addEventListener('change', () => Globe.setPointOfView(camera.position, Globe.position))
}

function onWindowResize() {
  // update globe size based on screen size
  const size = globeSize.find(s => window.innerWidth > s.breakpoint)?.size
  renderers.forEach(r => r.setSize(size, size))
}

function animate() {
  controls.update()
  renderers.forEach((r: any) => r.render(scene, camera))
  requestAnimationFrame(animate)
}
</script>

<template>
  <!-- positioning for test to see if it works -->
  <div ref="element" class="absolute lg:-right-96 md:-right-36 -right-20 xl:top-12 lg:top-28 md:-top-10 top-0 z-10" />
</template>
