<template>
  <section>
    <div>
      <div id="map" class="map" style="width: 100vw; height: 100vh" />
      asdf
    </div>
    <pre id="info" />
  </section>
</template>

<script>
import * as BABYLON from 'babylonjs'
import 'babylonjs-loaders'
import maplibregl from 'maplibre-gl'
// import * as THREE from 'three'
export default {
  name: 'NuxtTutorial',
  data: function () {
    return {
      map: null,
      modelMatrix: null,
      target: null,
      selectMesh: null,
      meshes: [],
      a: null
    }
  },
  mounted () {
    this.map = new maplibregl.Map({
      container: 'map', // container id
      // style: 'https://api.maptiler.com/maps/basic/style.json?key=get_your_own_OpIi9ZULNHzrESv6T2vL', // style URL
      style: 'https://api.maptiler.com/maps/streets/style.json?key=chtNHUeC692DJrDzNDcv',
      center: [127.12, 37.5], // starting position [lng, lat]
      maxZoom: 24,
      zoom: 18, // starting zoom
      pitch: 60
      // accessToken: 'pk.eyJ1IjoiZ29mbDI2IiwiYSI6ImNsZnEzbWNxdDBna2kzeGxmODhiNWVyaTkifQ.RtwKGiSvqC4IRLv7-OevFA'
      // antialias: true // create the gl context with MSAA antialiasing, so custom layers are antialiased
    })

    this.test()
  },
  methods: {
    test () {
      const map = this.map
      const modelOrigin = [127.12, 37.5]
      const modelAltitude = 0 // 위로 띄우기
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
      const modelMatrix = BABYLON.Matrix.Compose(
        new BABYLON.Vector3(modelTransform.scale, modelTransform.scale, modelTransform.scale),
        BABYLON.Quaternion.FromEulerAngles(modelRotate[0], modelRotate[1], modelRotate[2]),
        new BABYLON.Vector3(modelAsMercatorCoordinate.x, modelAsMercatorCoordinate.y, modelAsMercatorCoordinate.z)
      )
      let engine = null
      let scene = null
      let camera = null
      const thismeshes = this.meshes
      const customLayer = {
        id: '3d-model',
        type: 'custom',
        renderingMode: '3d',
        onAdd: async function (map, gl) {
          this.map = map
          engine = new BABYLON.Engine(gl, true, {
            useHighPrecisionMatrix: true
          }, true)
          scene = new BABYLON.Scene(engine)
          scene.autoClear = false
          scene.detachControl()
          scene.beforeRender = function () {
            engine.wipeCaches(true)
          }

          camera = new BABYLON.Camera('mapbox-camera', new BABYLON.Vector3(), scene)
          const light = new BABYLON.HemisphericLight('mapbox-light', BABYLON.Vector3.One(), scene)
          light.setEnabled(true)

          const { meshes } = await BABYLON.SceneLoader.ImportMeshAsync('', './aaaa/', 'cctvlight.glb', scene)
          console.info(meshes)
          meshes[0].scaling = new BABYLON.Vector3(1, 1, 1)
          meshes[0].position = new BABYLON.Vector3(0, 10, 0)
          for (let i = 0; i < meshes.length; i++) {
            meshes[i].metadata = {
              id: 'cctv'
            }
          }
          console.info(meshes)
          thismeshes.push(meshes)
        },
        render: function (gl, matrix) {
          const cameraMatrix = BABYLON.Matrix.FromArray(matrix)

          const mvpMatrix = modelMatrix.multiply(cameraMatrix)
          camera.freezeProjectionMatrix(mvpMatrix)

          scene.render(false)
          this.map.triggerRepaint()
        }
      }
      map.on('style.load', function () {
        map.addLayer(customLayer)
      })
      map.on('click', (e) => {
        const pixelCoords = e.point

        // Babylon.js의 Raycasting 기능을 사용하여 클릭된 매쉬를 찾음
        const pickInfo = scene.pick(pixelCoords.x, pixelCoords.y)

        // pickInfo가 null이 아니면 클릭된 매쉬가 있으므로 처리 가능
        if (pickInfo && pickInfo.hit && pickInfo.pickedMesh) {
          if (this.selectMesh) {
            console.info('1')
            thismeshes[0][2].material = this.a
          } else {
            // 클릭된 매쉬 처리
            const clickedMesh = pickInfo.pickedMesh
            console.info(clickedMesh)

            if (clickedMesh?.metadata?.id === 'cctv') {
              const yellowMat = new BABYLON.StandardMaterial('yellowMat')
              yellowMat.emissiveColor = new BABYLON.Color4(1, 0, 0)
              console.info(this.a)
              if (!this.a) {
                console.info(this.a)
                this.a = thismeshes[0][2].material
              }
              thismeshes[0][2].material = yellowMat
            }
            this.selectMesh = thismeshes
          }
          // 모든 자식 메시에 대한 전체 경계 상자 계산
          // let minPoint = new BABYLON.Vector3(Number.POSITIVE_INFINITY, Number.POSITIVE_INFINITY, Number.POSITIVE_INFINITY)
          // let maxPoint = new BABYLON.Vector3(Number.NEGATIVE_INFINITY, Number.NEGATIVE_INFINITY, Number.NEGATIVE_INFINITY)
          // const boundingInfo = clickedMesh.getBoundingInfo()
          // minPoint = BABYLON.Vector3.Minimize(minPoint, boundingInfo.boundingBox.minimumWorld)
          // maxPoint = BABYLON.Vector3.Maximize(maxPoint, boundingInfo.boundingBox.maximumWorld)

          // // x, y, z 크기 계산
          // const sizeX = maxPoint.x - minPoint.x
          // const sizeY = maxPoint.y - minPoint.y
          // const sizeZ = maxPoint.z - minPoint.z
          // // 크기가 sizeX, sizeY, sizeZ인 새로운 블록 생성
          // const newBlock = BABYLON.MeshBuilder.CreateBox('block', { width: sizeX, height: sizeY, depth: sizeZ, updatable: true }, scene)
          // console.info(sizeY)
          // // 새 블록의 위치 설정
          // newBlock.position = clickedMesh.getAbsolutePosition()
          // newBlock.position._y = sizeY / 2

          // // 새 블록에 투명 재질 적용
          // const material = new BABYLON.StandardMaterial('blockMaterial', scene)
          // // material.diffuseColor = new BABYLON.Color3(1, 0, 0)
          // material.alpha = 0.1 // 투명하게 설정
          // newBlock.material = material
          // this.selectMesh = newBlock
        }
      })
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
.mapboxgl-popup {
  max-width: 300px;
  font-family: 'Arial', sans-serif;
  font-size: 14px;
  line-height: 1.5;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
  background-color: #ffffff;
}

.mapboxgl-popup h3 {
  margin: 0 0 10px;
  font-weight: 700;
  font-size: 18px;
}

.mapboxgl-popup p {
  margin: 0;
}
</style>
