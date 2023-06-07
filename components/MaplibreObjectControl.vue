<template>
  <section>
    <div id="canvasZone">
      <div id="map" class="map" style="width: 100vw; height: 100vh">
        <!-- <canvas id="container" /> -->
      </div>
      <!-- <pre id="info" /> -->
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
      modelOrigin: [127.11880239063589, 37.50027974501366]
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

          const { meshes } = await BABYLON.SceneLoader.ImportMeshAsync('', './aaaa/', 'streetlamp2.glb', this.scene)
          meshes[0].scaling = new BABYLON.Vector3(1, 1, 1)
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
      // this.map.on('mousemove', function (e) {
      //   document.getElementById('info').innerHTML = JSON.stringify(e.point) + '<br />' + JSON.stringify(e.lngLat.wrap())
      // })
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#map { position: absolute; top: 0; bottom: 0; width: 100%; }
#info {
display: block;
position: relative;
margin: 0px auto;
width: 50%;
padding: 10px;
border: none;
border-radius: 3px;
font-size: 12px;
text-align: center;
color: #222;
background: #fff;
}
</style>
