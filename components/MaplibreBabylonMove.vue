<template>
  <section>
    <div>
      <div id="map" class="map" style="width: 100vw; height: 100vh" />
      asdf
    </div>
  </section>
</template>

<script>
import * as BABYLON from 'babylonjs'
import 'babylonjs-loaders'
import maplibregl from 'maplibre-gl'
// import * as GUI from 'babylonjs-gui'

export default {
  name: 'NuxtTutorial',
  data: function () {
    return {
      map: null,
      modelOrigin: [127.12, 37.5]
    }
  },
  mounted () {
    this.map = new maplibregl.Map({
      container: 'map', // container id
      // style: 'https://api.maptiler.com/maps/basic/style.json?key=get_your_own_OpIi9ZULNHzrESv6T2vL', // style URL
      style: 'https://api.maptiler.com/maps/streets/style.json?key=get_your_own_OpIi9ZULNHzrESv6T2vL',
      center: [127.12, 37.5], // starting position [lng, lat]
      maxZoom: 24,
      zoom: 18, // starting zoom
      pitch: 60,
      antialias: true // create the gl context with MSAA antialiasing, so custom layers are antialiased
    })

    this.test()
  },
  methods: {
    test () {
      // const modelOrigin = this.modelOrigin
      const modelAltitude = 0 // 위로 띄우기
      const modelRotate = [Math.PI / 2, 0, 0]

      let modelAsMercatorCoordinate = maplibregl.MercatorCoordinate.fromLngLat(
        this.modelOrigin,
        modelAltitude
      )

      let modelTransform = {
        translateX: modelAsMercatorCoordinate.x,
        translateY: modelAsMercatorCoordinate.y,
        translateZ: modelAsMercatorCoordinate.z,
        rotateX: modelRotate[0],
        rotateY: modelRotate[1],
        rotateZ: modelRotate[2],
        scale: modelAsMercatorCoordinate.meterInMercatorCoordinateUnits()
      }
      const _this = this
      let modelMatrix = BABYLON.Matrix.Compose(
        new BABYLON.Vector3(modelTransform.scale, modelTransform.scale, modelTransform.scale),
        BABYLON.Quaternion.FromEulerAngles(modelRotate[0], modelRotate[1], modelRotate[2]),
        new BABYLON.Vector3(modelAsMercatorCoordinate.x, modelAsMercatorCoordinate.y, modelAsMercatorCoordinate.z)
      )
      const customLayer = {
        id: '3d-model',
        type: 'custom',
        renderingMode: '3d',
        onAdd: async function (map, gl) {
          this.map = map
          const engine = new BABYLON.Engine(gl, true, {
            useHighPrecisionMatrix: true
          }, true)
          this.scene = new BABYLON.Scene(engine)
          this.scene.autoClear = false
          this.scene.detachControl()
          this.scene.beforeRender = function () {
            engine.wipeCaches(true)
          }
          this.camera = new BABYLON.Camera('mapbox-camera', new BABYLON.Vector3(), this.scene)
          const light = new BABYLON.HemisphericLight('mapbox-light', BABYLON.Vector3.One(), this.scene)
          light.setEnabled(true)

          const { meshes, animationGroups } = await BABYLON.SceneLoader.ImportMeshAsync('', './aaaa/', 'Soldier.glb', this.scene)
          // meshes[0].position = new BABYLON.Vector3(-20, 0, 0)
          meshes[0].scaling = new BABYLON.Vector3(30, 30, 30)
          meshes[0].rotation = new BABYLON.Vector3(0, 4, 0)
          meshes[0].actionManager = new BABYLON.ActionManager(this.scene)
          meshes[0].actionManager.registerAction(new BABYLON.ExecuteCodeAction(BABYLON.ActionManager.OnPickUpTrigger, function () {
            alert('sphere clicked')
          }))
          // mesh merge 안하고 해보기 merge 하면 애니메이션이 파괴되는 듯하다.
          console.info(this.scene)
          console.info(meshes)
          const model = this.scene.getMeshByName('__root__')

          const animWalk = new BABYLON.Animation('walkAnimation', 'position', 30, BABYLON.Animation.ANIMATIONTYPE_VECTOR3, BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE)
          const animRun = new BABYLON.Animation('runAnimation', 'position', 30, BABYLON.Animation.ANIMATIONTYPE_VECTOR3, BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE)
          const walkKeys = []
          const runKeys = []
          walkKeys.push({
            frame: 0,
            value: new BABYLON.Vector3(0, 0, 0)
          })

          walkKeys.push({
            frame: 120,
            value: new BABYLON.Vector3(100, 0, 100)
          })

          runKeys.push({
            frame: 0,
            value: new BABYLON.Vector3(0, 0, 0)
          })

          runKeys.push({
            frame: 80000,
            value: new BABYLON.Vector3(444400, 0, 444400)
          })

          animWalk.setKeys(walkKeys)
          animRun.setKeys(runKeys)
          model.animations = []
          model.animations.push(animWalk)
          model.animations.push(animRun)

          // 3D 모델 움직이기, 애니메이션 X, mesh merge 하는 로직
          // meshes.shift()

          // const target = BABYLON.Mesh.MergeMeshes(meshes, true, true, undefined, false, true)

          // const rotateFrames = []
          // const slideFrames = []
          // // const fadeFrames = []

          // const fps = 60

          // const rotateAnim = new BABYLON.Animation('rotateAnim', 'rotation.z', fps, BABYLON.Animation.ANIMATIONTYPE_FLOAT, BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE)
          // const slideAnim = new BABYLON.Animation('slideAnim', 'position', fps, BABYLON.Animation.ANIMATIONTYPE_VECTOR3, BABYLON.Animation.ANIMATIONLOOPMODE_CYCLE)
          // // const fadeAnim = new BABYLON.Animation('fadeAnim', 'visibility', fps, BABYLON.Animation.ANIMATIONTYPE_FLOAT, BABYLON.Animation.ANIMATIONLOOPMODE_CONSTANT)

          // rotateFrames.push({ frame: 0, value: 0 })
          // rotateFrames.push({ frame: 180, value: Math.PI / 2 })

          // slideFrames.push({ frame: 0, value: new BABYLON.Vector3(0, 30, 0) })
          // slideFrames.push({ frame: 45, value: new BABYLON.Vector3(-30, 20, 0) })
          // slideFrames.push({ frame: 90, value: new BABYLON.Vector3(0, 30, 0) })
          // slideFrames.push({ frame: 135, value: new BABYLON.Vector3(30, 20, 0) })
          // slideFrames.push({ frame: 180, value: new BABYLON.Vector3(0, 30, 0) })

          // // fadeFrames.push({ frame: 0, value: 1 })
          // // fadeFrames.push({ frame: 180, value: 0 })

          // rotateAnim.setKeys(rotateFrames)
          // slideAnim.setKeys(slideFrames)
          // // fadeAnim.setKeys(fadeFrames)

          // target.animations.push(rotateAnim)
          // target.animations.push(slideAnim)
          // // target.animations.push(fadeAnim)

          // const animControl = this.scene.beginDirectAnimation(target, [slideAnim], 0, 180, true, 1, () => {
          //   console.info('animation ended')
          //   target.setEnabled(false)
          // })

          const button1 = document.createElement('button')
          button1.style.top = '100px'
          button1.style.right = '30px'
          button1.textContent = '걷기'
          button1.style.width = '100px'
          button1.style.height = '100px'

          button1.setAttribute = ('id', 'but')
          button1.style.position = 'absolute'
          button1.style.color = 'black'

          document.body.appendChild(button1)

          button1.addEventListener('click', () => { // 밑에 주석 실행하려면 async 붙이기
            // await this.scene.beginDirectAnimation(target, [rotateAnim], 0, 180).waitAsync()
            // animControl.stop()

            // this.scene.beginAnimation(model, 0, 150, true)
            this.scene.beginDirectAnimation(model, [animWalk], 0, 120, true)
            animationGroups[3].play(true)
            animationGroups[1].pause()
            animationGroups[0].pause()
          })

          const button2 = document.createElement('button')
          button2.style.top = '200px'
          button2.style.right = '30px'
          button2.textContent = '뛰기'
          button2.style.width = '100px'
          button2.style.height = '100px'

          button2.setAttribute = ('id', 'but')
          button2.style.position = 'absolute'
          button2.style.color = 'black'

          document.body.appendChild(button2)

          button2.addEventListener('click', () => {
            this.scene.beginDirectAnimation(model, [animRun], 0, 80000, true)
            animationGroups[1].play(true)
            animationGroups[3].pause()
            animationGroups[0].pause()
          })

          const button3 = document.createElement('button')
          button3.style.top = '300px'
          button3.style.right = '30px'
          button3.textContent = '멈춰'
          button3.style.width = '100px'
          button3.style.height = '100px'

          button3.setAttribute = ('id', 'but')
          button3.style.position = 'absolute'
          button3.style.color = 'black'

          document.body.appendChild(button3)

          button3.addEventListener('click', () => {
            this.scene.stopAllAnimations()
            animationGroups[0].play(true)
            animationGroups[3].pause()
            animationGroups[1].pause()
          })

          const button4 = document.createElement('button')
          button4.style.top = '300px'
          button4.style.left = '30px'
          button4.textContent = '이동'
          button4.style.width = '100px'
          button4.style.height = '100px'

          button4.setAttribute = ('id', 'but')
          button4.style.position = 'absolute'
          button4.style.color = 'black'

          document.body.appendChild(button4)

          button4.addEventListener('click', () => {
            this.map.once('click', function (e) {
              document.addEventListener('click', () => {
                _this.modelOrigin[0] = Number(e.lngLat.lng)
                _this.modelOrigin[1] = Number(e.lngLat.lat)
                modelAsMercatorCoordinate = maplibregl.MercatorCoordinate.fromLngLat(
                  _this.modelOrigin,
                  modelAltitude
                )
                modelTransform = {
                  translateX: modelAsMercatorCoordinate.x,
                  translateY: modelAsMercatorCoordinate.y,
                  translateZ: modelAsMercatorCoordinate.z,
                  rotateX: modelRotate[0],
                  rotateY: modelRotate[1],
                  rotateZ: modelRotate[2],
                  scale: modelAsMercatorCoordinate.meterInMercatorCoordinateUnits()
                }
                modelMatrix = BABYLON.Matrix.Compose(
                  new BABYLON.Vector3(modelTransform.scale, modelTransform.scale, modelTransform.scale),
                  BABYLON.Quaternion.FromEulerAngles(modelRotate[0], modelRotate[1], modelRotate[2]),
                  new BABYLON.Vector3(modelAsMercatorCoordinate.x, modelAsMercatorCoordinate.y, modelAsMercatorCoordinate.z)
                )
                console.info(_this.modelOrigin)
              }, { once: true })
            })
          })
        },
        render: function (gl, matrix) {
          const cameraMatrix = BABYLON.Matrix.FromArray(matrix)

          const mvpMatrix = modelMatrix.multiply(cameraMatrix)
          this.camera.freezeProjectionMatrix(mvpMatrix)

          this.scene.render(false)
          this.map.triggerRepaint()
        }
      }
      this.map.on('style.load', function () {
        _this.map.addLayer(customLayer)
      })
    }

  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#map { position: absolute; top: 0; bottom: 0; width: 100%; }

</style>
