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

      const { meshes, animationGroups } = await BABYLON.SceneLoader.ImportMeshAsync('', './aaaa/', 'ankylosaurus2.glb', scene)
      console.info(meshes)
      meshes[0].scaling = new BABYLON.Vector3(0.5, 0.5, 0.5)
      meshes[0].actionManager = new BABYLON.ActionManager(scene)
      meshes[0].actionManager.registerAction(new BABYLON.ExecuteCodeAction(BABYLON.ActionManager.OnPickUpTrigger, function () {
        alert('sphere clicked')
      }))

      // const texture = new BABYLON.Texture('./aaaa/material/ankylosaurus_col2.tga', scene)
      // texture.wrapU = BABYLON.Texture.CLAMP_ADDRESSMODE
      // texture.wrapV = BABYLON.Texture.CLAMP_ADDRESSMODE
      // texture.uScale = 1
      // texture.vScale = 1

      // const material = new BABYLON.StandardMaterial('material', scene)
      // material.diffuseTexture = texture
      // console.info(material)
      // // 이제 material을 mesh에 적용하면 됩니다.
      // meshes[1].material = material
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
      console.info(animationGroups)
      // let currentAnimationIndex = 0
      // animationGroups[2].play()
      // animationGroups[3].play()
      // button1.addEventListener('click', () => {
      //   // 현재 재생 중인 애니메이션을 정지합니다.
      //   if (animationGroups[currentAnimationIndex]) {
      //     animationGroups[currentAnimationIndex].stop()
      //   }

      //   // 다음 애니메이션 그룹을 재생합니다.
      //   currentAnimationIndex++
      //   if (currentAnimationIndex >= animationGroups.length) {
      //     currentAnimationIndex = 0
      //   }

      //   animationGroups[currentAnimationIndex].play(true)
      //   console.info(currentAnimationIndex)
      // })
      return scene
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#container { position: absolute; top: 0; bottom: 0; width: 100%; height: 1000px}
</style>
