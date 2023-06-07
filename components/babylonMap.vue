<template>
  <section>
    <canvas id="container" />
  </section>
</template>
<script>
import * as BABYLON from 'babylonjs'
import 'babylonjs-loaders'

export default {
  data: () => {
    return {
      canvas: null,
      engine: null,
      scene: null,
      importResult: null
    }
  },
  async mounted () {
    this.canvas = document.getElementById('container')
    this.engine = new BABYLON.Engine(this.canvas, true)

    const sceneToRender = await this.createScene()

    this.engine.runRenderLoop(function () {
      sceneToRender.render()
    })
  },
  methods: {
    async createScene () {
      const scene = new BABYLON.Scene(this.engine)
      scene.clearColor = new BABYLON.Color3.Black()
      const alpha = Math.PI / 4
      const beta = Math.PI / 3
      const radius = 4
      const target = new BABYLON.Vector3(0, 0, 0)

      const camera = new BABYLON.ArcRotateCamera('Camera', alpha, beta, radius, target, scene)
      camera.attachControl(this.canvas, true)

      const light = new BABYLON.HemisphericLight('light', new BABYLON.Vector3(1, 1, 0))
      light.setEnabled(true)
      const skybox = BABYLON.MeshBuilder.CreateBox('skyBox', { size: 1000.0 }, scene)
      const skyboxMaterial = new BABYLON.StandardMaterial('skyBox', scene)
      skyboxMaterial.backFaceCulling = false
      skyboxMaterial.reflectionTexture = new BABYLON.CubeTexture('https://playground.babylonjs.com/textures/skybox', scene)
      skyboxMaterial.reflectionTexture.coordinatesMode = BABYLON.Texture.SKYBOX_MODE
      skyboxMaterial.diffuseColor = new BABYLON.Color3(0, 0, 0)
      skyboxMaterial.specularColor = new BABYLON.Color3(0, 0, 0)
      skybox.material = skyboxMaterial
      const result = await BABYLON.SceneLoader.ImportMeshAsync('', './aaaa/', 'oh8.gltf', scene)
      result.meshes[0].scaling = new BABYLON.Vector3(0.1, 0.1, 0.1)
      result.meshes[0].scaling.x *= -1
      console.info(result)
      return scene
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#container { position: absolute; top: 0; bottom: 0; width: 100%; height: 1000px}
</style>
