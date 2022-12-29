<template>
  <section>
    <div>
      <div id="map" class="map" style="width: 100vw; height: 100vh" />
    </div>
  </section>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { DragControls } from 'three/examples/jsm/controls/DragControls.js'
// import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import maplibregl from 'maplibre-gl'
// import 'mapbox-gl/dist/mapbox-gl.css'

export default {
  name: 'NuxtTutorial',
  data: function () {
    return {
      map: null,
      enableSelection: false
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
    this.init()
  },
  methods: {
    init () {
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
        /* Since our 3D model is in real world meters, a scale transform needs to be
* applied since the CustomLayerInterface expects units in MercatorCoordinates.
*/
        scale: modelAsMercatorCoordinate.meterInMercatorCoordinateUnits()
      }
      const _this = this
      let camera = null
      let controls = null
      let scene = null
      const objects = []
      const group = new THREE.Group()
      const mouse = new THREE.Vector2()
      const raycaster = new THREE.Raycaster()
      const customLayer = {
        id: '3d-model',
        type: 'custom',
        renderingMode: '3d',
        onAdd: function (map, gl) {
          console.info(this)
          camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 5000)
          scene = new THREE.Scene()

          // create two three.js lights to illuminate the model
          const directionalLight = new THREE.DirectionalLight(0xFFFFFF)
          directionalLight.position.set(0, -70, 100).normalize()
          scene.add(directionalLight)

          const directionalLight2 = new THREE.DirectionalLight(0xFFFFFF)
          directionalLight2.position.set(0, 70, 100).normalize()
          scene.add(directionalLight2)

          // use the three.js GLTF loader to add the 3D model to the three.js scene

          scene.add(group)
          const loader = new GLTFLoader()
          loader.load(
            'exhome_gl1.gltf',
            function (gltf) {
              group.add(gltf.scene)
              objects.push(gltf.scene)
            }
          )
          this.renderMap = map

          // use the MapLibre GL JS map canvas for three.js
          this.renderer = new THREE.WebGLRenderer({
            canvas: map.getCanvas(),
            context: gl,
            antialias: true
          })

          this.renderer.autoClear = false
          controls = new DragControls([...objects], camera, this.renderer.domElement)
          console.info(controls)
          controls.addEventListener('drag', () => {
            console.info('1')
            this.renderer.render(scene, camera)
          })
          document.addEventListener('click', onClick)
          window.addEventListener('mousedown', _this.onKeyDown)
          window.addEventListener('mouseup', _this.onKeyUp)

          const __this = this
          function onClick (event) {
            console.info(event)
            event.preventDefault()
            if (_this.enableSelection === true) {
              const draggableObjects = controls.getObjects()
              draggableObjects.length = 0

              mouse.x = (event.clientX / window.innerWidth) * 2 - 1
              mouse.y = -(event.clientY / window.innerHeight) * 2 + 1

              raycaster.setFromCamera(mouse, camera)

              const intersections = raycaster.intersectObjects(objects, true)

              if (intersections.length > 0) {
                const object = intersections[0].object

                if (group.children.includes(object) === true) {
                  object.material.emissive.set(0x000000)
                  scene.attach(object)
                } else {
                  object.material.emissive.set(0xAAAAAA)
                  group.attach(object)
                }

                controls.transformGroup = true
                draggableObjects.push(group)
              }

              if (group.children.length === 0) {
                controls.transformGroup = false
                draggableObjects.push(...objects)
              }
            }

            __this.render()
          }
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
          if (matrix) {
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

            camera.projectionMatrix = m.multiply(l)
          }
          this.renderer.state.reset()
          this.renderer.render(scene, camera)
          this.renderMap.triggerRepaint()
        }
      }
      this.map.on('load', function () {
        _this.map.addLayer(customLayer)
        _this.map.on('click', function (e) {
          console.info('123')
        })
      })
    },
    onKeyDown (event) {
      console.info('누르기', event)
      this.enableSelection = (event.buttons === 1)
    },
    onKeyUp (event) {
      console.info('때기', event)
      this.enableSelection = false
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#map { position: absolute; top: 0; bottom: 0; width: 100%; }

</style>
