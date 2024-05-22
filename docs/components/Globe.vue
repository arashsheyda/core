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
let renderers, Globe, scene, camera, controls

const marker = '<svg viewBox="0 0 512 512" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M281.44 397.667H438.32C443.326 397.667 448.118 395.908 452.453 393.427C456.789 390.946 461.258 387.831 463.76 383.533C466.262 379.236 468.002 374.36 468 369.399C467.998 364.437 466.266 359.563 463.76 355.268L357.76 172.947C355.258 168.65 352.201 165.534 347.867 163.053C343.532 160.573 337.325 158.813 332.32 158.813C327.315 158.813 322.521 160.573 318.187 163.053C313.852 165.534 310.795 168.65 308.293 172.947L281.44 219.587L227.733 129.13C225.229 124.834 222.176 120.307 217.84 117.827C213.504 115.346 208.713 115 203.707 115C198.701 115 193.909 115.346 189.573 117.827C185.238 120.307 180.771 124.834 178.267 129.13L46.8267 355.268C44.3208 359.563 44.0022 364.437 44 369.399C43.9978 374.36 44.3246 379.235 46.8267 383.533C49.3288 387.83 53.7979 390.946 58.1333 393.427C62.4688 395.908 67.2603 397.667 72.2667 397.667H171.2C210.401 397.667 238.934 380.082 258.827 346.787L306.88 263.4L332.32 219.587L410.053 352.44H306.88L281.44 397.667ZM169.787 352.44H100.533L203.707 174.36L256 263.4L221.361 323.784C208.151 345.387 193.089 352.44 169.787 352.44Z" fill="#00DC82"/></svg>'
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
      r.setPixelRatio(window.devicePixelRatio / 2)
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
      // markers
      .htmlElementsData(arcsData.flatMap(arc => [
        { lat: arc.startLat, lng: arc.startLng },
        { lat: arc.endLat, lng: arc.endLng },
      ]))
      .htmlElement(() => {
        const el = document.createElement('div')
        el.innerHTML = marker
        el.style.width = '20px'
        return el
      })
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

  // const WireFrameGlobe = new ThreeGlobe({
  //   waitForGlobeReady: true,
  //   animateIn: true,
  // })

  // const wireFrameMaterial = WireFrameGlobe.globeMaterial()
  // wireFrameMaterial.wireframe = true
  // wireFrameMaterial.color = new Color(0x00DC82)

  // wireFrameMaterial.transparent = true
  // wireFrameMaterial.opacity = 0.3

  // scene.add(WireFrameGlobe)

  // Update pov when camera moves
  Globe.setPointOfView(camera.position, Globe.position)
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
