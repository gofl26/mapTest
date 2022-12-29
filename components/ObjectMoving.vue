<template>
  <section>
    <div id="container" />
  </section>
</template>
<script>
import * as THREE from 'three'
// import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
import { DragControls } from 'three/examples/jsm/controls/DragControls.js'
// import Stats from 'three/examples/jsm/libs/stats.module.js'
// import maplibregl from 'maplibre-gl'
// import 'mapbox-gl/dist/mapbox-gl.css'

export default {
  name: 'NuxtTutorial',
  data: function () {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      dragControls: null,
      mixer: null,
      clock: null,
      group: null,
      openAction: null,
      closeAction: null,
      actions: null,
      settings: null,
      mouse: null,
      raycaster: null,
      enableSelection: null,
      objects: [],
      crossFadeControls: []
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      // 3d모델을 띄울 div 선택
      const container = document.getElementById('container')
      document.body.appendChild(container)
      this.clock = new THREE.Clock()

      // 카메라 세팅
      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000)
      this.camera.position.set(0, 0, 1000)

      // 마우스 세팅
      this.mouse = new THREE.Vector2()
      this.raycaster = new THREE.Raycaster()

      // 씬 세팅
      this.scene = new THREE.Scene()
      const BACKGROUND_COLOR = 0xF1F1F1
      this.scene.background = new THREE.Color(BACKGROUND_COLOR)

      // 물체에 빛 추가 그림자 x
      const hemiLight = new THREE.HemisphereLight(0xFFFFFF, 0xFFFFFF, 0.61)
      hemiLight.position.set(0, 50, 0)
      this.scene.add(hemiLight)

      // 씬에 빛 추가 그림자 o
      const dirLight = new THREE.DirectionalLight(0xFFFFFF, 0.54)
      dirLight.position.set(-8, 12, 8)
      dirLight.castShadow = true
      dirLight.shadow.mapSize = new THREE.Vector2(1024, 1024)
      this.scene.add(dirLight)

      // 땅 추가
      const mesh = new THREE.Mesh(new THREE.PlaneGeometry(100, 100), new THREE.MeshPhongMaterial({ color: 0x999999, depthWrite: false }))
      mesh.rotation.x = -Math.PI / 2
      mesh.receiveShadow = true
      this.scene.add(mesh)

      this.group = new THREE.Group()
      this.scene.add(this.group)

      const geometry = new THREE.BoxGeometry(40, 40, 40)
      this.objects = []
      for (let i = 0; i < 200; i++) {
        const object = new THREE.Mesh(geometry, new THREE.MeshLambertMaterial({ color: Math.random() * 0xFFFFFF }))

        object.position.x = Math.random() * 1000 - 500
        object.position.y = Math.random() * 600 - 300
        object.position.z = Math.random() * 800 - 400

        object.rotation.x = Math.random() * 2 * Math.PI
        object.rotation.y = Math.random() * 2 * Math.PI
        object.rotation.z = Math.random() * 2 * Math.PI

        object.scale.x = Math.random() * 2 + 1
        object.scale.y = Math.random() * 2 + 1
        object.scale.z = Math.random() * 2 + 1

        object.castShadow = true
        object.receiveShadow = true

        this.scene.add(object)

        this.objects.push(object)
      }

      // render 작업
      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.renderer.outputEncoding = THREE.sRGBEncoding
      this.renderer.shadowMap.enabled = true
      container.appendChild(this.renderer.domElement)

      this.dragControls = new DragControls([...this.objects], this.camera, this.renderer.domElement)
      this.dragControls.addEventListener('drag', () => {
        // if (this.controls.enabled) { this.controls.enabled = false }
        // console.info(this.controls.enabled)
        this.renderer.render(this.scene, this.camera)
      })

      window.addEventListener('resize', this.onWindowResize)
      document.addEventListener('click', this.onClick)
      document.addEventListener('keydown', this.onKeyDown)
      document.addEventListener('keyup', this.onKeyUp)
      document.addEventListener('mouseup', this.onMouseUp)
      this.renderer.render(this.scene, this.camera)

      // 옵저빙 세팅
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.update()

      // stats ??
      // this.stats = new Stats()
      // container.appendChild(this.stats.dom)
    },
    onWindowResize () {
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()

      this.renderer.setSize(window.innerWidth, window.innerHeight)

      this.renderer.render(this.scene, this.camera)
    },
    onMouseUp () {
      this.controls.enabled = true
      this.controls.update()
      this.renderer.render(this.scene, this.camera)
    },
    onKeyUp () {
      this.enableSelection = false
    },
    onKeyDown (event) {
      this.enableSelection = (event.keyCode === 16)
    },
    onClick (event) {
      event.preventDefault()
      if (this.enableSelection === true) {
        const draggableObjects = this.dragControls.getObjects()
        draggableObjects.length = 0

        this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1
        this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1

        this.raycaster.setFromCamera(this.mouse, this.camera)

        const intersections = this.raycaster.intersectObjects(this.objects, true)

        if (intersections.length > 0) {
          const object = intersections[0].object

          if (this.group.children.includes(object) === true) {
            object.material.emissive.set(0x000000)
            this.scene.attach(object)
          } else {
            object.material.emissive.set(0xAAAAAA)
            this.group.attach(object)
          }

          this.dragControls.transformGroup = true
          draggableObjects.push(this.group)
        }

        if (this.group.children.length === 0) {
          this.dragControls.transformGroup = false
          draggableObjects.push(...this.objects)
        }
      }
      this.renderer.render(this.scene, this.camera)
    },

    threeDAnimation () {
      // Render loop
      requestAnimationFrame(this.threeDAnimation)

      // this.openWeight = this.openAction.getEffectiveWeight()

      // panel 업데이트
      // this.updateWeightSliders()

      // this.updateCrossFadeControls()

      // 시간 설정
      // this.controls.update()
      this.renderer.render(this.scene, this.camera)
    },
    createPanel () {
      const panel = new GUI({ width: 310 })

      const folder4 = panel.addFolder('Crossfading')
      const folder5 = panel.addFolder('Blend Weights')

      const _this = this
      this.settings = {
        'from walk to idle': function () {
          _this.prepareCrossFade(_this.openAction, 1.0)
        },
        'modify idle weight': 0.0,
        'use default duration': true,
        'set custom duration': 3.5
      }

      this.crossFadeControls.push(folder4.add(this.settings, 'from walk to idle'))

      folder5.add(this.settings, 'modify idle weight', 0.0, 1.0, 0.01).listen().onChange(function (weight) {
        this.setWeight(this.openAction, weight)
      })

      folder4.open()
      folder5.open()
    },
    prepareCrossFade (startAction, endAction, defaultDuration) {
      // Switch default / custom crossfade duration (according to the user's choice)

      const duration = this.setCrossFadeDuration(defaultDuration)

      // If the current action is 'idle' (duration 4 sec), execute the crossfade immediately;
      // else wait until the current action has finished its current loop

      this.executeCrossFade(startAction, endAction, duration)
    },
    setCrossFadeDuration (defaultDuration) {
      // Switch default crossfade duration <-> custom crossfade duration

      if (this.settings['use default duration']) {
        return defaultDuration
      } else {
        return this.settings['set custom duration']
      }
    },
    executeCrossFade (startAction, endAction, duration) {
      // Not only the start action, but also the end action must get a weight of 1 before fading
      // (concerning the start action this is already guaranteed in this place)

      this.setWeight(endAction, 1)
      endAction.time = 0

      // Crossfade with warping - you can also try without warping by setting the third parameter to false

      startAction.crossFadeTo(endAction, duration, true)
    },
    setWeight (action, weight) {
      // This function is needed, since animationAction.crossFadeTo() disables its start action and sets
      // the start action's timeScale to ((start animation's duration) / (end animation's duration))
      action.enabled = true
      action.setEffectiveTimeScale(1)
      action.setEffectiveWeight(weight)
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
