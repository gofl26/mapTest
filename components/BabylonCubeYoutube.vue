<template>
  <section>
    <div id="canvasZone">
      <canvas id="container" />
    </div>
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
      sceneToRender: null
    }
  },
  async mounted () {
    await this.initFunction()
  },
  methods: {
    attachHtml (id, W, H, L, T, p, ts) {
      let el = document.createElement('div')
      let exist = false
      if (document.getElementById('spDiv' + id)) {
        exist = true
        el = document.getElementById('spDiv' + id)
      }
      const zone = document.getElementById('canvasZone')
      const w = zone.offsetWidth.valueOf() * 1.0
      const h = zone.offsetHeight.valueOf() * 1.0
      el.setAttribute('style', 'transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg) rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);perspective:' + p + 'position:absolute;top:' + (h / 2 - H / 2 + T) + 'px;left:' + (w / 2 - W / 2 + L) + 'px;z-index:100;background-color:#ffffff;width:' + W + 'px;height:' + H + 'px;opacity:1.;')
      // el.setAttribute('style', 'transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg) rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);transform-origin: 50% 50%;perspective: ' + p + ';-webkit-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg) rotateY(' + ts.ry + 'deg)  rotateZ(' + ts.rz + 'deg);-webkit-transform-origin: 50% 50%;-moz-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);-moz-transform-origin: 50% 50%;-o-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);-o-transform-origin: 50% 50%;-ms-transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg) rotateZ(' + ts.rz + 'deg);-ms-transform-origin:50% 50%;transform: scale(1.0) scaleZ(1.0) rotateX(' + ts.rx + 'deg)  rotateY(' + ts.ry + 'deg)  rotateZ(' + ts.rz + 'deg);transform-origin: 50% 50%;position:absolute;top:' + (h / 2 - H / 2 + T) + 'px;left:' + (w / 2 - W / 2 + L) + 'px;z-index:100;background-color:#ffffff;width:' + W + 'px;height:' + H + 'px;opacity:1.;')
      el.id = 'spDiv' + id
      // zone.setAttribute('style', '-o-perspective: ' + p + '-o-perspective-origin: 50% 50%; position:relative')
      zone.setAttribute('style', '-o-perspective: ' + p + ';-o-perspective-origin: 50% 50%;-webkit-perspective: ' + p + ';-webkit-perspective-origin: 50% 50%;-moz-perspective: ' + p + ';-moz-perspective-origin: 50% 50%;-ms-perspective: ' + p + ';-ms-perspective-origin: 50% 50%;perspective: ' + p + ';perspective-origin: 50% 50%;position:relative')
      if (!exist) {
        el.innerHTML = '<div> 왜 안되냐고ㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗㅗ</div><iframe width="560" height="315" src="https://www.youtube.com/embed/wPDl_XuUGqY?autoplay=1&mute=1" title="YouTube video player" frameborder="0" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe><img src="http://i.imgur.com/RCkw1Ae.gif" width="100%" height="100%" />'
        zone.appendChild(el)
      }
      // <iframe width="560" height="315" src="https://www.youtube.com/embed/wPDl_XuUGqY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    },
    createPage (scene, camera, url = '6y4BMVhkQPs') {
      const _this = this
      scene.registerBeforeRender(function () {
        const cp = camera.position
        const dx = Math.atan(Math.sqrt(cp.x * cp.x + cp.z * cp.z) / cp.y) * 180 / Math.PI
        const dy = 0
        let sign = 1
        const baseSize = 180
        if (dx > 0) { sign = -1 }
        const dz = sign * Math.atan(cp.x / cp.z) * 180.0 / Math.PI
        const size = baseSize * camera.fov / Math.sqrt(Math.pow(cp.x, 2.0) + Math.pow(cp.y, 2.0) + Math.pow(cp.z, 2.0))
        _this.attachHtml('PageWeb', baseSize * size, baseSize * size, 0, 0, '100px', { rx: dx, ry: dy, rz: dz })
      })
    },
    async initFunction () {
      this.canvas = document.getElementById('container')
      this.engine = await this.asyncEngineCreation()
      this.startRenderLoop()
      this.scene = this.createScene()
      this.sceneToRender = this.scene
    },
    createScene () {
      const scene = new BABYLON.Scene(this.engine)

      const camera = new BABYLON.ArcRotateCamera('camera1', 1, 2, 20, new BABYLON.Vector3(0, 3000, 0.0), scene)
      camera.setTarget(BABYLON.Vector3.Zero())
      camera.attachControl(this.canvas, true)

      const light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene)
      light.intensity = 0.7

      this.createPage(scene, camera)

      return scene
    },
    startRenderLoop () {
      this.engine.runRenderLoop(() => {
        if (this.sceneToRender && this.sceneToRender.activeCamera) {
          this.sceneToRender.render()
        }
      })
    },
    async createDefaultEngine () { return await new BABYLON.Engine(this.canvas, true, { preserveDrawingBuffer: true, stencil: true, disableWebGL2Support: true }) },
    async asyncEngineCreation () {
      try {
        return await this.createDefaultEngine()
      } catch (e) {
        console.log('the available createEngine function failed. Creating the default engine instead')
        return await this.createDefaultEngine()
      }
    }
  }
}
</script>
<style>
body { margin: 0; padding: 0; }
#container { position: absolute; top: 0; bottom: 0; width: 100%; height: 1000px}
</style>
