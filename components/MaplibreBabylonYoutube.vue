<template>
  <section>
    <div id="canvasZone">
      <div id="map" class="map" style="width: 100vw; height: 100vh">
        <!-- <canvas id="container" /> -->
      </div>
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
      style: 'https://api.maptiler.com/maps/streets/style.json?key=chtNHUeC692DJrDzNDcv',
      // style: null,
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

      const modelAsMercatorCoordinate = maplibregl.MercatorCoordinate.fromLngLat(
        this.modelOrigin,
        modelAltitude
      )

      const modelTransform = {
        translateX: modelAsMercatorCoordinate.x,
        translateY: modelAsMercatorCoordinate.y,
        translateZ: modelAsMercatorCoordinate.z,
        rotateX: modelRotate[0],
        rotateY: modelRotate[1],
        rotateZ: modelRotate[2],
        scale: modelAsMercatorCoordinate.meterInMercatorCoordinateUnits()
      }
      const _this = this
      const modelMatrix = BABYLON.Matrix.Compose(
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
          let canvas = null
          let engine = null
          let scene = null
          let sceneToRender = null
          let camera = null
          async function createDefaultEngine () {
            return await new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true, disableWebGL2Support: true })
          }
          async function asyncEngineCreation () {
            try {
              return await createDefaultEngine()
            } catch (e) {
              console.log('the available createEngine function failed. Creating the default engine instead')
              return await createDefaultEngine()
            }
          }
          function startRenderLoop () {
            engine.runRenderLoop(() => {
              if (sceneToRender && sceneToRender.activeCamera) {
                sceneToRender.render()
              }
            })
          }
          function attachHtml (id, W, H, L, T, p, ts) {
            let el = document.createElement('div')
            let exist = false
            if (document.getElementById('spDiv' + id)) {
              exist = true
              el = document.getElementById('spDiv' + id)
            }
            const zone = document.getElementById('canvasZone')
            const w = zone.offsetWidth.valueOf() * 1.0
            const h = zone.offsetHeight.valueOf() * 1.0

            el.setAttribute('style', 'transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg) rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);transform-origin: 50% 50%;perspective: ' + p + ';-webkit-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg) rotateY(' + ts.ry + 'deg)  rotateZ(' + ts.rz + 'deg);-webkit-transform-origin: 50% 50%;-moz-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);-moz-transform-origin: 50% 50%;-o-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);-o-transform-origin: 50% 50%;-ms-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);-ms-transform-origin:50% 50%;transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg)  rotateZ(' + ts.rz + 'deg);transform-origin: 50% 50%;position:absolute;top:' + (h / 2 - H / 2 + T) + 'px;left:' + (w / 2 - W / 2 + L) + 'px;z-index:100;background-color:#ffffff;width:' + W + 'px;height:' + H + 'px;opacity:1.;')
            el.id = 'spDiv' + id
            zone.setAttribute('style', '-o-perspective: ' + p + ';-o-perspective-origin: 50% 50%;-webkit-perspective: ' + p + ';-webkit-perspective-origin: 50% 50%;-moz-perspective: ' + p + ';-moz-perspective-origin: 50% 50%;-ms-perspective: ' + p + ';-ms-perspective-origin: 50% 50%;perspective: ' + p + ';perspective-origin: 50% 50%;position:relative')
            if (!exist) {
              el.innerHTML = '<div> 왜 안되냐고ㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗ</div><iframe width="560" height="315" src="https://www.youtube.com/embed/wPDl_XuUGqY?autoplay=1&mute=1" title="YouTube video player" frameborder="0" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe><img src="http://i.imgur.com/RCkw1Ae.gif" width="100%" height="100%" />'
              zone.appendChild(el)
            }
            // <iframe width="560" height="315" src="https://www.youtube.com/embed/wPDl_XuUGqY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
          }
          function createPage (scene, camera, url = '6y4BMVhkQPs') {
            scene.registerBeforeRender(function () {
              const cp = camera.position
              const dx = Math.atan(Math.sqrt(cp.x * cp.x + cp.z * cp.z) / cp.y) * 180 / Math.PI
              const dy = 0
              let sign = 1
              const baseSize = 180
              if (dx > 0) { sign = -1 }
              const dz = sign * Math.atan(cp.x / cp.z) * 180.0 / Math.PI
              const size = baseSize * camera.fov / Math.sqrt(Math.pow(cp.x, 2.0) + Math.pow(cp.y, 2.0) + Math.pow(cp.z, 2.0))
              attachHtml('PageWeb', baseSize * size, baseSize * size, 0, 0, '800px', { rx: dx, ry: dy, rz: dz })
            })
          }
          function createScene () {
            scene = new BABYLON.Scene(engine)

            const camera = new BABYLON.ArcRotateCamera('camera1', 1, 2, 20, new BABYLON.Vector3(0, 150, 0.0), scene)
            camera.setTarget(BABYLON.Vector3.Zero())
            camera.attachControl(canvas, true)

            const light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene)
            light.intensity = 0.7

            createPage(scene, camera)

            return scene
          }
          async function initFunction () {
            canvas = document.createElement('canvas')
            // canvas = document.getElementById('container')
            engine = await asyncEngineCreation()
            startRenderLoop()
            scene = createScene()
            sceneToRender = scene
          }
          await initFunction()
          camera = new BABYLON.Camera('mapbox-camera', new BABYLON.Vector3(), scene)
          this.camera = camera
          const light = new BABYLON.HemisphericLight('mapbox-light', BABYLON.Vector3.One(), scene)
          light.setEnabled(true)
          this.scene = scene
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
