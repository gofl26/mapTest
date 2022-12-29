<template>
  <section>
    <div>
      <div id="map" class="map" style="width: 100vw; height: 100vh" />
      asdf
      <nav class="listing-group">
        <input id="scrollZoom" v-model="scrollZoom" type="checkbox">
        <label for="scrollZoom">scroll Zoom</label>
        <input id="boxZoom" v-model="boxZoom" type="checkbox">
        <label for="boxZoom">Box zoom</label>
        <input id="dragRotate" v-model="dragRotate" type="checkbox">
        <label for="dragRotate">Drag rotate</label>
        <input id="dragPan" v-model="dragPan" type="checkbox">
        <label for="dragPan">Drag pan</label>
        <input id="keyboard" v-model="keyboard" type="checkbox">
        <label for="keyboard">Keyboard</label>
        <input id="doubleClickZoom" v-model="doubleClickZoom" type="checkbox">
        <label for="doubleClickZoom">Double click zoom</label>
        <input id="touchZoomRotate" v-model="touchZoomRotate" type="checkbox">
        <label for="touchZoomRotate">Touch zoom rotate</label>
        <button @click="fly">
          날아라
        </button>
      </nav>
    </div>
  </section>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
// import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import maplibregl from 'maplibre-gl'
// import 'mapbox-gl/dist/mapbox-gl.css'

export default {
  name: 'NuxtTutorial',
  data: function () {
    return {
      scrollZoom: true,
      boxZoom: true,
      dragRotate: true,
      dragPan: true,
      keyboard: true,
      doubleClickZoom: true,
      touchZoomRotate: true,
      renderMap: null,
      map: null,
      scaleX: 3,
      scaleY: 3,
      scaleZ: 3
    }
  },
  watch: {
    scrollZoom: function (newVal) {
      if (newVal) { this.map.scrollZoom.enable() } else { this.map.scrollZoom.disable() }
    },
    boxZoom: function (newVal) {
      if (newVal) { this.map.boxZoom.enable() } else { this.map.boxZoom.disable() }
    },
    keyboard: function (newVal) {
      if (newVal) { this.map.keyboard.enable() } else { this.map.keyboard.disable() }
    },
    touchZoomRotate: function (newVal) {
      if (newVal) { this.map.touchZoomRotate.enable() } else { this.map.touchZoomRotate.disable() }
    },
    doubleClickZoom: function (newVal) {
      if (newVal) { this.map.doubleClickZoom.enable() } else { this.map.doubleClickZoom.disable() }
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
    asdff () {
      console.info('1')
    },
    test () {
      const modelOrigin = [127.12, 37.5]
      const modelAltitude = 0
      const modelRotate = [Math.PI / 2, 0, 0]

      const modelAsMercatorCoordinate = maplibregl.MercatorCoordinate.fromLngLat(
        modelOrigin,
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
      const customLayer = {
        id: '3d-model',
        type: 'custom',
        renderingMode: '3d',
        onAdd: function (map, gl) {
          this.camera = new THREE.Camera()
          this.scene = new THREE.Scene()

          const camera = new THREE.PerspectiveCamera(10, window.innerWidth / window.innerHeight, 0.1, 2000)

          this.scene.add(camera)
          const hemiLight = new THREE.HemisphereLight(0xFFFFFF, 0xFFFFFF, 1)
          hemiLight.position.set(0, 50, 0)
          this.scene.add(hemiLight)
          // this.scene.add(group)
          const dirLight = new THREE.DirectionalLight(0xFFFFFF, 0.54)
          dirLight.position.set(-8, 12, 8)
          dirLight.castShadow = true
          dirLight.shadow.mapSize = new THREE.Vector2(1024, 1024)
          this.scene.add(dirLight)

          // use the three.js GLTF loader to add the 3D model to the three.js scene
          const loader = new GLTFLoader()
          // loader.load(
          //   'eiffel.glb',
          //   (gltf) => {
          //     const setting = gltf.scene
          //     setting.scale.x = 2000
          //     setting.scale.y = 2000
          //     setting.scale.z = 2000
          //     this.scene.add(setting)
          //   }
          // )

          loader.load(
            'exhome_gl1.gltf',
            (gltf) => {
              console.info(_this)
              const setting = gltf.scene
              setting.scale.x = _this.scaleX
              setting.scale.y = _this.scaleY
              setting.scale.z = _this.scaleZ
              // setting.position.x = 60
              // setting.position.y = 40
              // setting.position.z = -60
              setting.rotation.y = 142
              // setting.rotation.x = 300
              // setting.rotation.z = -90
              // gltf.scene.traverse(function (node) {
              // })
              this.scene.add(setting)
            }
          )
          this.renderMap = map
          // this.map = map
          // use the Mapbox GL JS map canvas for three.js
          this.renderer = new THREE.WebGLRenderer({
            canvas: map.getCanvas(),
            context: gl,
            antialias: true
          })

          this.renderer.autoClear = false
        },
        render: function (gl, matrix) {
          const rotationX = new THREE.Matrix4().makeRotationAxis(
            new THREE.Vector3(1, 0, 0),
            modelTransform.rotateX
          )
          const rotationY = new THREE.Matrix4().makeRotationAxis(
            new THREE.Vector3(0, 1, 0),
            modelTransform.rotateY
          )
          const rotationZ = new THREE.Matrix4().makeRotationAxis(
            new THREE.Vector3(0, 0, 1),
            modelTransform.rotateZ
          )

          const m = new THREE.Matrix4().fromArray(matrix)
          const l = new THREE.Matrix4()
            .makeTranslation(
              modelTransform.translateX,
              modelTransform.translateY,
              modelTransform.translateZ
            )
            .scale(
              new THREE.Vector3(
                modelTransform.scale,
                -modelTransform.scale,
                modelTransform.scale
              )
            )
            .multiply(rotationX)
            .multiply(rotationY)
            .multiply(rotationZ)

          this.camera.projectionMatrix = m.multiply(l)
          this.renderer.resetState()
          this.renderer.render(this.scene, this.camera)

          this.renderMap.triggerRepaint()
          // this.map.triggerRepaint()
        }
      }
      this.map.on('load', () => {
        this.map.addLayer(customLayer)
      })
    },
    fly () {
      // this.map.zoomTo(19, { duration: 9000 }) /// 현재 시점에서 줌인
      this.map.flyTo({
        bearing: 90,
        center: [127.12, 37.5],
        zoom: 23,
        pitch: 85
      })// 해당 위경도로 날아가기
      this.scrollZoom = false
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#map { position: absolute; top: 0; bottom: 0; width: 100%; }

.put-box {
font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
font-weight: 600;
position: absolute;
top: 10px;
left: 10px;
z-index: 1;
border-radius: 3px;
max-width: 10%;
color: #fff;
}

.listing-group {
font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
font-weight: 600;
position: absolute;
top: 10px;
right: 10px;
z-index: 1;
border-radius: 3px;
max-width: 20%;
color: #fff;
}

.listing-group input[type='checkbox']:first-child + label {
border-radius: 3px 3px 0 0;
}
.listing-group label:last-child {
border-radius: 0 0 3px 3px;
border: none;
}

.listing-group input[type='checkbox'] {
display: none;
}

.listing-group input[type='checkbox'] + label {
background-color: #3386c0;
display: block;
cursor: pointer;
padding: 10px;
border-bottom: 1px solid rgba(0, 0, 0, 0.25);
}

.listing-group input[type='checkbox'] + label {
background-color: #3386c0;
text-transform: capitalize;
}

.listing-group input[type='checkbox'] + label:hover,
.listing-group input[type='checkbox']:checked + label {
background-color: #4ea0da;
}

.listing-group input[type='checkbox']:checked + label:before {
content: '✔';
margin-right: 5px;
}
</style>
