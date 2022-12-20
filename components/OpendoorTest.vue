<template>
  <section>
    <div id="container" />
  </section>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
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
      mixer: null,
      clock: null,
      openAction: null,
      closeAction: null,
      actions: null,
      settings: null,
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
      this.clock = new THREE.Clock()

      // 카메라 세팅
      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000)
      this.camera.position.set(0, 0, 10)

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

      // 3D 모델 추가
      const loader = new GLTFLoader()
      const _this = this
      loader.load('animated_old_door.glb', function (gltf) {
        console.info(gltf)
        const setting = gltf.scene
        // 스케일 주기
        setting.scale.x = 0.01
        setting.scale.y = 0.01
        setting.scale.z = 0.01

        // 회전 주기
        // setting.rotation.x = -2.6

        // 위치 주기
        // setting.position.x = 42

        // 그림자 효과 주기
        setting.traverse((o) => {
          if (o.isMesh) {
            o.castShadow = true
            o.receiveShadow = true
          }
        })
        _this.scene.add(setting)

        // 패널 생성
        _this.createPanel()

        /// 애니메이션 효과
        _this.mixer = new THREE.AnimationMixer(gltf.scene)
        const animations = gltf.animations
        const openAnimation = animations[0]

        // closeAni.duration = 2.4
        console.info(openAnimation)
        _this.openAction = _this.mixer.clipAction(openAnimation).setLoop(THREE.LoopOnce)
        _this.openAction.time = 1.2
        _this.openAction.clampWhenFinished = true

        console.info(_this.openAction)
        _this.actions = [_this.openAction]

        _this.activateAllActions()
        _this.threeDAnimation()
      })

      // render 작업
      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.renderer.outputEncoding = THREE.sRGBEncoding
      this.renderer.shadowMap.enabled = true
      container.appendChild(this.renderer.domElement)

      // 옵저빙 세팅
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.update()

      // stats ??
      // this.stats = new Stats()
      // container.appendChild(this.stats.dom)
    },
    threeDAnimation () {
      // Render loop
      requestAnimationFrame(this.threeDAnimation)

      // this.openWeight = this.openAction.getEffectiveWeight()

      // panel 업데이트
      // this.updateWeightSliders()

      // this.updateCrossFadeControls()

      // 시간 설정
      const mixerUpdateDelta = this.clock.getDelta()
      this.mixer.update(mixerUpdateDelta)
      this.controls.update()
      this.renderer.render(this.scene, this.camera)
    },
    activateAllActions () {
      // this.setWeight(this.openAction, this.settings['modify idle weight'])

      this.actions.forEach(function (action) {
        console.info(action)
        action.play()
      })
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
