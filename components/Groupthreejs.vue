<template>
  <div>
    <div id="info">
      <input id="animationStart" v-model="animationStart" type="checkbox">
      <label for="animationStart">멈춰!</label>
    </div>
    <div id="info2">
      <input id="animationChange" v-model="animationChange" type="checkbox">
      <label for="animationChange">딴거 해봐!</label>
    </div>
    <!-- <div id="map" class="map" style="width: 100vw; height: 100vh" /> -->
    <canvas id="canvas" style="width: 100vw; height: 100vh" />
  </div>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

// import maplibregl from 'maplibre-gl'
// import 'mapbox-gl/dist/mapbox-gl.css'

export default {
  name: 'NuxtTutorial',
  data: function () {
    return {
      scene: null,
      camera: null,
      renderer: null,
      cube: null,
      controls: null,
      mixer: null,
      clock: null,
      animationStart: false,
      animationChange: false
    }
  },
  mounted () {
    // this.drawCube()
    // this.drawLine()
    this.threeDModel()
  },
  methods: {
    threeDModel () {
      this.clock = new THREE.Clock()
      this.renderer = new THREE.WebGLRenderer({
        canvas: document.querySelector('#canvas'),
        antialias: true
      })
      this.renderer.shadowMap.enabled = true

      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      document.body.appendChild(this.renderer.domElement)
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        2000
      )
      // 카메라의 위치 설정 (x: 0, y: 0, z: 5)
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.camera.position.set(0, 0, 10)
      this.controls.update()
      this.scene = new THREE.Scene()
      const BACKGROUND_COLOR = 0xF1F1F1
      this.scene.background = new THREE.Color(BACKGROUND_COLOR)
      const loader = new GLTFLoader()

      const _this = this
      loader.load('exhome_gl1.gltf', function (gltf) {
        console.info(gltf)
        // gltf.scene.children[0].children[0].children[0].children[0].children.forEach((o, index) => {
        //   console.info(o.name, index)
        //   const setting = o
        //   setting.rotation.x = Math.PI / 2
        //   setting.rotation.z = Math.PI
        //   setting.rotation.y = Math.PI
        //   setting.position.x = 3 * (index + 1)
        //   setting.position.y = 2 * (index + 1)
        //   _this.scene.add(setting)
        //   _this.renderer.render(_this.scene, _this.camera)
        // })
        const setting = gltf.scene
        setting.scale.x = 1
        setting.scale.y = 1
        setting.scale.z = 1
        _this.scene.add(setting)
        _this.renderer.render(_this.scene, _this.camera)
        setting.traverse((o) => {
          if (o.isMesh) {
            o.castShadow = true

            o.receiveShadow = true
          }
        })
        /// 애니메이션 효과
        // let action = null
        _this.mixer = new THREE.AnimationMixer(gltf.scene)
        // action = _this.mixer.clipAction(gltf.animations[0])
        // action.play()
        _this.threeDAnimation()
      })

      this.renderer.outputEncoding = THREE.sRGBEncoding
      const hemiLight = new THREE.HemisphereLight(0xFFFFFF, 0xFFFFFF, 0.61)
      hemiLight.position.set(0, 50, 0)

      // Add hemisphere light to scene
      this.scene.add(hemiLight)
      const dirLight = new THREE.DirectionalLight(0xFFFFFF, 0.54)
      dirLight.position.set(-8, 12, 8)
      dirLight.castShadow = true
      dirLight.shadow.mapSize = new THREE.Vector2(1024, 1024)

      /// 바닥생성
      const floorGeometry = new THREE.PlaneGeometry(5000, 5000, 1, 1)

      const floorMaterial = new THREE.MeshPhongMaterial({

        color: 0x999999,

        shininess: 0

      })

      const floor = new THREE.Mesh(floorGeometry, floorMaterial)

      floor.rotation.x = -0.5 * Math.PI

      floor.receiveShadow = true

      floor.position.y = -0.05

      this.scene.add(floor)
      // Add directional Light to scene
      this.scene.add(dirLight)
    },
    threeDAnimation () {
      requestAnimationFrame(this.threeDAnimation)
      /// 멈춰!
      let delta = null
      if (this.animationStart) { delta = null } else {
        delta = this.clock.getDelta()
      }
      this.mixer.update(delta)
      this.controls.update()
      this.renderer.render(this.scene, this.camera)
    },
    drawLine () {
      this.renderer = new THREE.WebGLRenderer()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      document.body.appendChild(this.renderer.domElement)

      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 500)
      this.camera.position.set(0, 0, 100)
      this.camera.lookAt(0, 0, 0)

      this.scene = new THREE.Scene()
      const material = new THREE.LineBasicMaterial({ color: 0x0000FF })
      const points = []
      points.push(new THREE.Vector3(-10, 0, 0))
      points.push(new THREE.Vector3(0, 10, 0))
      points.push(new THREE.Vector3(10, 0, 0))

      const geometry = new THREE.BufferGeometry().setFromPoints(points)

      const line = new THREE.Line(geometry, material)

      this.scene.add(line)
      this.renderer.render(this.scene, this.camera)
    },
    drawCube () {
      this.scene = new THREE.Scene()
      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 500)

      this.renderer = new THREE.WebGLRenderer()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      document.body.appendChild(this.renderer.domElement)

      const geometry = new THREE.BoxGeometry(1, 1, 1)
      const material = new THREE.MeshBasicMaterial({ color: 0x00FF00 })
      this.cube = new THREE.Mesh(geometry, material)
      this.scene.add(this.cube)

      this.camera.position.z = 5
      this.cubeAnimation()
    },
    cubeAnimation () {
      requestAnimationFrame(this.cubeAnimation)
      this.cube.rotation.x += 0.01
      this.cube.rotation.y += 0.01
      this.renderer.render(this.scene, this.camera)
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#map { position: absolute; top: 0; bottom: 0; width: 100%; }

#info { position: absolute; top: 10px; width: 100%; text-align: center; z-index: 100; background-color: white; display:block; }

#info2 { position: absolute; top: 30px; width: 100%; text-align: center; z-index: 100; background-color: white; display:block; }
</style>
