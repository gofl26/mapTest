<template>
  <section>
    <div id="container" />
  </section>
</template>

<script>

import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

// import Stats from 'three/examples/jsm/libs/stats.module.js'
// import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'

export default {
  data: function () {
    return {
      scene: null,
      renderer: null,
      camera: null,
      controls: null,
      mixer: null,
      clock: null,
      bathAction: null,
      deadAction: null,
      drinkAction: null,
      eatAction: null,
      glitchAction: null,
      gosleepAction: null,
      idleAction: null,
      sitAction: null,
      sleepAction: null,
      walkAction: null
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
      loader.load('floppa_rtx_on.glb', function (gltf) {
        console.info(gltf)
        const setting = gltf.scene
        // 스케일 주기
        setting.scale.x = 1
        setting.scale.y = 1
        setting.scale.z = 1

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
        // _this.createPanel()

        /// 애니메이션 효과
        const animations = gltf.animations
        _this.mixer = new THREE.AnimationMixer(gltf.scene)

        _this.bathAction = _this.mixer.clipAction(animations[0])
        _this.deadAction = _this.mixer.clipAction(animations[1])
        _this.drinkAction = _this.mixer.clipAction(animations[2])
        _this.eatAction = _this.mixer.clipAction(animations[3])
        _this.glitchAction = _this.mixer.clipAction(animations[4])
        _this.gosleepAction = _this.mixer.clipAction(animations[5])
        _this.idleAction = _this.mixer.clipAction(animations[6])
        _this.sitAction = _this.mixer.clipAction(animations[7])
        _this.sleepAction = _this.mixer.clipAction(animations[8])
        _this.walkAction = _this.mixer.clipAction(animations[9])

        // _this.activateAllActions()
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
    }
  }
}
</script>

<style>

</style>
