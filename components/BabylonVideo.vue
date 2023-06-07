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
  mounted () {
    this.canvas = document.getElementById('container')
    this.engine = new BABYLON.Engine(this.canvas, true)

    const sceneToRender = this.createScene()

    this.engine.runRenderLoop(function () {
      sceneToRender.render()
    })
  },
  methods: {
    createScene () {
      // This creates a basic Babylon Scene object (non-mesh)
      const scene = new BABYLON.Scene(this.engine)

      // This creates and positions a free camera (non-mesh)
      const camera = new BABYLON.ArcRotateCamera('arcR', -Math.PI / 2, Math.PI / 2, 15, BABYLON.Vector3.Zero(), scene)

      // This attaches the camera to the canvas
      camera.attachControl(this.canvas, true)

      const planeOpts = {
        height: 15.4762,
        width: 17.3967,
        sideOrientation: BABYLON.Mesh.DOUBLESIDE
      }
      const ANote0Video = BABYLON.MeshBuilder.CreatePlane('plane', planeOpts, scene)
      const vidPos = (new BABYLON.Vector3(0, 0, 0.1))
      ANote0Video.position = vidPos
      const ANote0VideoMat = new BABYLON.StandardMaterial('m', scene)
      const ANote0VideoVidTex = new BABYLON.VideoTexture('', 'aaaa/mp4sample.mp4', scene)
      ANote0VideoMat.diffuseTexture = ANote0VideoVidTex
      ANote0VideoMat.roughness = 1
      ANote0VideoMat.emissiveColor = new BABYLON.Color3.White()
      ANote0Video.material = ANote0VideoMat
      scene.onPointerObservable.add(function (evt) {
        if (evt.pickInfo.pickedMesh === ANote0Video) {
          // console.log("picked");
          if (ANote0VideoVidTex.video.paused) { ANote0VideoVidTex.video.play() } else { ANote0VideoVidTex.video.pause() }
          console.log(ANote0VideoVidTex.video.paused ? 'paused' : 'playing')
        }
      }, BABYLON.PointerEventTypes.POINTERPICK)
      // console.log(ANote0Video);
      return scene
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#container { position: absolute; top: 0; bottom: 0; width: 100%; height: 1000px}
</style>
