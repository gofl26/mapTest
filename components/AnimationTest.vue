<template>
  <section>
    <div id="container" />
  </section>
</template>

<script>

import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

import Stats from 'three/examples/jsm/libs/stats.module.js'
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'

export default {
  data: function () {
    return {
      scene: null,
      renderer: null,
      camera: null,
      stats: null,
      controls: null,
      model: null,
      skeleton: null,
      mixer: null,
      clock: null,
      idleAction: null,
      walkAction: null,
      runAction: null,
      idleWeight: null,
      walkWeight: null,
      runWeight: null,
      actions: null,
      crossFadeControls: [],
      settings: null,
      singleStepMode: false,
      sizeOfNextStep: 0
    }
  },
  mounted () {
    const container = document.getElementById('container')

    this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000)

    this.camera.position.set(1, 2, -3)
    this.camera.lookAt(0, 1, 0)

    this.clock = new THREE.Clock()

    this.scene = new THREE.Scene()
    this.scene.background = new THREE.Color(0xA0A0A0)
    this.scene.fog = new THREE.Fog(0xA0A0A0, 10, 50)

    const hemiLight = new THREE.HemisphereLight(0xFFFFFF, 0x444444)
    hemiLight.position.set(0, 20, 0)
    this.scene.add(hemiLight)

    const dirLight = new THREE.DirectionalLight(0xFFFFFF)
    dirLight.position.set(-3, 10, -10)
    dirLight.castShadow = true
    dirLight.shadow.camera.top = 2
    dirLight.shadow.camera.bottom = -2
    dirLight.shadow.camera.left = -2
    dirLight.shadow.camera.right = 2
    dirLight.shadow.camera.near = 0.1
    dirLight.shadow.camera.far = 40
    this.scene.add(dirLight)

    // scene.add( new THREE.CameraHelper( dirLight.shadow.camera ) );

    // ground

    const mesh = new THREE.Mesh(new THREE.PlaneGeometry(100, 100), new THREE.MeshPhongMaterial({ color: 0x999999, depthWrite: false }))
    mesh.rotation.x = -Math.PI / 2
    mesh.receiveShadow = true
    this.scene.add(mesh)

    const loader = new GLTFLoader()
    const _this = this
    loader.load('Soldier.glb', function (gltf) {
      _this.model = gltf.scene
      _this.scene.add(_this.model)
      console.info(gltf)
      _this.model.traverse(function (object) {
        if (object.isMesh) { object.castShadow = true }
      })

      //

      _this.skeleton = new THREE.SkeletonHelper(_this.model)
      _this.skeleton.visible = false
      _this.scene.add(_this.skeleton)

      //

      _this.createPanel()

      //

      const animations = gltf.animations

      _this.mixer = new THREE.AnimationMixer(_this.model)

      _this.idleAction = _this.mixer.clipAction(animations[0])
      _this.walkAction = _this.mixer.clipAction(animations[3])
      _this.runAction = _this.mixer.clipAction(animations[1])

      _this.actions = [_this.idleAction, _this.walkAction, _this.runAction]

      _this.activateAllActions()

      _this.animate()
    })

    this.renderer = new THREE.WebGLRenderer({ antialias: true })
    this.renderer.setPixelRatio(window.devicePixelRatio)
    this.renderer.setSize(window.innerWidth, window.innerHeight)
    this.renderer.outputEncoding = THREE.sRGBEncoding
    this.renderer.shadowMap.enabled = true
    container.appendChild(this.renderer.domElement)
    this.controls = new OrbitControls(this.camera, this.renderer.domElement)
    this.controls.update()
    this.stats = new Stats()
    container.appendChild(this.stats.dom)

    window.addEventListener('resize', this.onWindowResize)
  },
  methods: {
    showSkeleton (visibility) {
      this.skeleton.visible = visibility
    },
    modifyTimeScale (speed) {
      this.mixer.timeScale = speed
    },
    deactivateAllActions () {
      this.actions.forEach(function (action) {
        action.stop()
      })
    },
    activateAllActions () {
      this.setWeight(this.idleAction, this.settings['modify idle weight'])
      this.setWeight(this.walkAction, this.settings['modify walk weight'])
      this.setWeight(this.runAction, this.settings['modify run weight'])

      this.actions.forEach(function (action) {
        action.play()
      })
    },
    pauseContinue () {
      if (this.singleStepMode) {
        this.singleStepMode = false
        this.unPauseAllActions()
      } else if (this.idleAction.paused) {
        this.unPauseAllActions()
      } else {
        this.pauseAllActions()
      }
    },
    pauseAllActions () {
      this.actions.forEach(function (action) {
        action.paused = true
      })
    },
    unPauseAllActions () {
      this.actions.forEach(function (action) {
        action.paused = false
      })
    },
    toSingleStepMode () {
      this.unPauseAllActions()

      this.singleStepMode = true
      this.sizeOfNextStep = this.settings['modify step size']
    },
    prepareCrossFade (startAction, endAction, defaultDuration) {
      // Switch default / custom crossfade duration (according to the user's choice)

      const duration = this.setCrossFadeDuration(defaultDuration)

      // Make sure that we don't go on in singleStepMode, and that all actions are unpaused

      this.singleStepMode = false
      this.unPauseAllActions()

      // If the current action is 'idle' (duration 4 sec), execute the crossfade immediately;
      // else wait until the current action has finished its current loop

      if (startAction === this.idleAction) {
        this.executeCrossFade(startAction, endAction, duration)
      } else {
        this.synchronizeCrossFade(startAction, endAction, duration)
      }
    },
    setCrossFadeDuration (defaultDuration) {
      // Switch default crossfade duration <-> custom crossfade duration

      if (this.settings['use default duration']) {
        return defaultDuration
      } else {
        return this.settings['set custom duration']
      }
    },
    synchronizeCrossFade (startAction, endAction, duration) {
      this.mixer.addEventListener('loop', onLoopFinished)

      function onLoopFinished (event) {
        if (event.action === startAction) {
          this.mixer.removeEventListener('loop', onLoopFinished)

          this.executeCrossFade(startAction, endAction, duration)
        }
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
    },
    updateWeightSliders () {
      // Called by the render loop
      this.settings['modify idle weight'] = this.idleWeight
      this.settings['modify walk weight'] = this.walkWeight
      this.settings['modify run weight'] = this.runWeight
    },
    updateCrossFadeControls () {
      // Called by the render loop
      if (this.idleWeight === 1 && this.walkWeight === 0 && this.runWeight === 0) {
        this.crossFadeControls[0].disable()
        this.crossFadeControls[1].enable()
        this.crossFadeControls[2].disable()
        this.crossFadeControls[3].disable()
      }

      if (this.idleWeight === 0 && this.walkWeight === 1 && this.runWeight === 0) {
        this.crossFadeControls[0].enable()
        this.crossFadeControls[1].disable()
        this.crossFadeControls[2].enable()
        this.crossFadeControls[3].disable()
      }

      if (this.idleWeight === 0 && this.walkWeight === 0 && this.runWeight === 1) {
        this.crossFadeControls[0].disable()
        this.crossFadeControls[1].disable()
        this.crossFadeControls[2].disable()
        this.crossFadeControls[3].enable()
      }
    },
    onWindowResize () {
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()

      this.renderer.setSize(window.innerWidth, window.innerHeight)
    },
    animate () {
      // Render loop

      requestAnimationFrame(this.animate)

      this.idleWeight = this.idleAction.getEffectiveWeight()
      this.walkWeight = this.walkAction.getEffectiveWeight()
      this.runWeight = this.runAction.getEffectiveWeight()

      // Update the panel values if weights are modified from "outside" (by crossfadings)

      this.updateWeightSliders()

      // Enable/disable crossfade controls according to current weight values

      this.updateCrossFadeControls()

      // Get the time elapsed since the last frame, used for mixer update (if not in single step mode)

      let mixerUpdateDelta = this.clock.getDelta()

      // If in single step mode, make one step and then do nothing (until the user clicks again)

      if (this.singleStepMode) {
        mixerUpdateDelta = this.sizeOfNextStep
        this.sizeOfNextStep = 0
      }

      // Update the animation mixer, the stats panel, and render this frame
      this.mixer.update(mixerUpdateDelta)

      this.controls.update()
      this.stats.update()
      this.renderer.render(this.scene, this.camera)
    },
    showModel (visibility) {
      this.model.visible = visibility
    },
    createPanel () {
      const panel = new GUI({ width: 310 })

      const folder1 = panel.addFolder('Visibility')
      const folder2 = panel.addFolder('Activation/Deactivation')
      const folder3 = panel.addFolder('Pausing/Stepping')
      const folder4 = panel.addFolder('Crossfading')
      const folder5 = panel.addFolder('Blend Weights')
      const folder6 = panel.addFolder('General Speed')

      this.settings = {
        'show model': true,
        'show skeleton': false,
        'deactivate all': this.deactivateAllActions,
        'activate all': this.activateAllActions,
        'pause/continue': this.pauseContinue,
        'make single step': this.toSingleStepMode,
        'modify step size': 0.05,
        'from walk to idle': function () {
          this.prepareCrossFade(this.walkAction, this.idleAction, 1.0)
        },
        'from idle to walk': function () {
          this.prepareCrossFade(this.idleAction, this.walkAction, 0.5)
        },
        'from walk to run': function () {
          this.prepareCrossFade(this.walkAction, this.runAction, 2.5)
        },
        'from run to walk': function () {
          this.prepareCrossFade(this.runAction, this.walkAction, 5.0)
        },
        'use default duration': true,
        'set custom duration': 3.5,
        'modify idle weight': 0.0,
        'modify walk weight': 1.0,
        'modify run weight': 0.0,
        'modify time scale': 1.0
      }

      folder1.add(this.settings, 'show model').onChange(this.showModel)
      folder1.add(this.settings, 'show skeleton').onChange(this.showSkeleton)
      folder2.add(this.settings, 'deactivate all')
      folder2.add(this.settings, 'activate all')
      folder3.add(this.settings, 'pause/continue')
      folder3.add(this.settings, 'make single step')
      folder3.add(this.settings, 'modify step size', 0.01, 0.1, 0.001)
      this.crossFadeControls.push(folder4.add(this.settings, 'from walk to idle'))
      this.crossFadeControls.push(folder4.add(this.settings, 'from idle to walk'))
      this.crossFadeControls.push(folder4.add(this.settings, 'from walk to run'))
      this.crossFadeControls.push(folder4.add(this.settings, 'from run to walk'))
      folder4.add(this.settings, 'use default duration')
      folder4.add(this.settings, 'set custom duration', 0, 10, 0.01)
      folder5.add(this.settings, 'modify idle weight', 0.0, 1.0, 0.01).listen().onChange(function (weight) {
        this.setWeight(this.idleAction, weight)
      })
      folder5.add(this.settings, 'modify walk weight', 0.0, 1.0, 0.01).listen().onChange(function (weight) {
        this.setWeight(this.walkAction, weight)
      })
      folder5.add(this.settings, 'modify run weight', 0.0, 1.0, 0.01).listen().onChange(function (weight) {
        this.setWeight(this.runAction, weight)
      })
      folder6.add(this.settings, 'modify time scale', 0.0, 1.5, 0.01).onChange(this.modifyTimeScale)

      folder1.open()
      folder2.open()
      folder3.open()
      folder4.open()
      folder5.open()
      folder6.open()
    }
  }
}
</script>

<style>

</style>
