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

      const result = await BABYLON.SceneLoader.ImportMeshAsync('', './aaaa/', 'Soldier.glb', scene)
      this.importResult = result
      BABYLON.Mesh.CreateGround('ground1', 6, 6, 2, scene)

      // GUI
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

      button1.addEventListener('click', () => {
        result.animationGroups[3].play(true)
        result.animationGroups[1].pause()
        result.animationGroups[0].pause()
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
        result.animationGroups[1].play(true)
        result.animationGroups[3].pause()
        result.animationGroups[0].pause()
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
        result.animationGroups[0].play(true)
        result.animationGroups[3].pause()
        result.animationGroups[1].pause()
      })

      return scene
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#container { position: absolute; top: 0; bottom: 0; width: 100%; height: 1000px}
</style>
