<template>
  <section>
    <canvas id="container" />
  </section>
</template>
<script>
import * as BABYLON from 'babylonjs'

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
    /// /////// 블랜더를 이용해서 fbx로 변환하는 식으로 하는게 제일 나은 방법같음
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

      BABYLON.Mesh.CreateGround('ground1', 6, 6, 2, scene)

      const result = await BABYLON.SceneLoader.ImportMeshAsync('', './Dinosaurs/', 'ankylosaurus.fbx', scene)
      this.importResult = result
      return scene
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#container { position: absolute; top: 0; bottom: 0; width: 100%; height: 1000px}
</style>
