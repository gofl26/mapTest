<!-- Please remove this file from your project -->
<template>
  <div>
    <div id="map" class="map" style="width: 100vw; height: 100vh" />
    asdf
  </div>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
// import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import mapboxgl from 'mapbox-gl'
import 'mapbox-gl/dist/mapbox-gl.css'

export default {
  name: 'NuxtTutorial',
  data: function () {
    return {

    }
  },
  mounted () {
    this.test()
  },
  methods: {
    test () {
      mapboxgl.accessToken = 'pk.eyJ1IjoiZm5qZWxzMDkiLCJhIjoiY2xiZnprY2Q2MGIxMjN1bXB3ODJmeGdheiJ9.PG0HKXyL0ux1ShP8-efbbA'
      const map = new mapboxgl.Map({
        container: 'map', // container ID
        style: 'mapbox://styles/mapbox/streets-v12', // style URL
        center: [127.1, 37.5], // starting position [lng, lat]
        zoom: 16 // starting zoom
      })

      const modelOrigin = [127.12, 37.5]
      const modelAltitude = 0
      const modelRotate = [Math.PI / 2, 0, 0]

      const modelAsMercatorCoordinate = mapboxgl.MercatorCoordinate.fromLngLat(
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

      const customLayer = {
        id: '3d-model',
        type: 'custom',
        renderingMode: '3d',
        onAdd: function (map, gl) {
          this.camera = new THREE.Camera()
          this.scene = new THREE.Scene()

          const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 2000)
          camera.position.set(0, 0, 5)
          this.scene.add(camera)

          // create two three.js lights to illuminate the model
          const directionalLight = new THREE.DirectionalLight(0xFFFFFF)
          directionalLight.position.set(0, -70, 100).normalize()
          this.scene.add(directionalLight)

          const directionalLight2 = new THREE.DirectionalLight(0xFFFFFF)
          directionalLight2.position.set(0, 70, 100).normalize()
          this.scene.add(directionalLight2)

          // use the three.js GLTF loader to add the 3D model to the three.js scene
          const loader = new GLTFLoader()
          loader.load(
            'eiffel.glb',
            (gltf) => {
              const setting = gltf.scene
              setting.scale.x = 1000
              setting.scale.y = 1000
              setting.scale.z = 1000
              // setting.rotation.y = -2.6
              // setting.position.x = 42
              // setting.position.y = 40
              // setting.position.z = -50
              console.info(gltf.scene)
              // setting.rotation.x = 300
              // setting.position.y = -150
              // gltf.scene.traverse(function (node) {
              // })
              this.scene.add(setting)
            }
          )

          this.map = map
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
          this.map.triggerRepaint()
        }
      }

      map.on('style.load', () => {
        map.addLayer(customLayer, 'waterway-label')
      })
      console.info(map)
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#map { position: absolute; top: 0; bottom: 0; width: 100%; }
</style>
