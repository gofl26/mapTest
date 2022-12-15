<template>
  <section>
    <div id="map" class="map" style="width: 100vw; height: 100vh">
      <div class="mapboxgl-ctrl-top-left">
        <div id="toolsControl" class="tools-i mapboxgl-ctrl mapboxgl-ctrl-group btn-group btn-group-toggle" style="margin-left:120px;" data-toggle="buttons">
          <!--<button type="button" name="ALL" id="ALL" title="All" alt="All"><i class="fas fa-globe-americas"></i></button>-->
          <button id="NYC" type="button" name="NYC" title="Statue of Liberty, NYC" alt="Statue of Liberty, NYC">
            <i style="width: 26px; height: 26px; background: url(./images/test.jpg) no-repeat center center; background-size: contain; display: block;" />
          </button>
          <button id="PAR" type="button" name="PAR" title="Eiffel Tower, Paris" alt="Eiffel Tower, Paris">
            <i style="width: 26px; height: 26px; background: url(./images/test.jpg) no-repeat center center; background-size: contain; display: block;" />
          </button>
        </div>
      </div>
    </div>
    <div id="hour" class="mapboxgl-map" />
    <input id="time" type="range" min="0" max="86400">
  </section>
</template>

<script>
// import * as THREE from 'three'
// import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import mapboxgl from 'mapbox-gl'
import { Threebox } from 'threebox-plugin'
import Stats from 'three/examples/jsm/libs/stats.module.js'

// import Pulse from './Pulses/Pulse'

// const pulse = new Pulse(400, 4000, true)

export default {
  name: 'ThreeboxExample',
  data () {
    return {
      accessToken:
        'pk.eyJ1IjoiZ29mbDI2IiwiYSI6ImNsYm14OHRxbDBtMGgzcG55ZmV6Nm12YngifQ.y5Z6JcWu3hvN6YmKO5jfQQ',
      minZoom: 12,
      seconds: 0,
      date: new Date(),
      stats: new Stats(),
      mapConfig: {}
    }
  },
  mounted () {
    mapboxgl.accessToken = this.accessToken

    this.mapConfig = {
      // ALL: { center: [-100.021884, 39.609738, 0], zoom: 3.25, pitch: 12, bearing: 0, timezone: 'US/Central' },
      NYC: {
        origin: [127.12, 37.5, 0], center: [127.12, 37.5, 0], zoom: 17.7, pitch: 76, bearing: -10, scale: { x: 1053, y: 1657.294, z: 1655 }, timezone: 'America/New_York'
      },
      PAR: {
        origin: [127.12, 37.5, 0], center: [127.12, 37.5, 0], zoom: 15.95, pitch: 76, bearing: 0, scale: { x: 5621.06, y: 6480.4, z: 5621.06 }, timezone: 'Europe/Paris'
      },
      names: {
        customLayer: 'custom-layer',
        fillExtrusion: 'composite-b',
        fillExtrusionLayer: 'building-b'
      }
    }
    const point = this.mapConfig.NYC
    const p = document.querySelector('#hour')

    const styles = {
      day: 'satellite-v9',
      night: 'dark-v10'
    }
    const selectedStyle = styles.day

    const map = new mapboxgl.Map({
      style: 'mapbox://styles/mapbox/' + selectedStyle, // 'mapbox://styles/mapbox/satellite-v9',
      center: point.center,
      zoom: point.zoom,
      pitch: point.pitch,
      bearing: point.bearing,
      container: 'map',
      antialias: true,
      hash: true
    })

    map.addControl(new mapboxgl.NavigationControl())
    // Add geolocate control to the map.
    map.addControl(
      new mapboxgl.GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true
        },
        trackUserLocation: true
      })
    )

    // we can add Threebox to mapbox to add built-in mouseover/mouseout and click behaviors
    window.tb = new Threebox(
      map,
      map.getCanvas().getContext('webgl'),
      {
        realSunlight: true,
        sky: true,
        enableSelectingObjects: true, // change this to false to disable 3D objects selection
        enableTooltips: true // change this to false to disable default tooltips on fill-extrusion and 3D models
      }
    )

    let time = this.date.getHours() * 60 * 60 + this.date.getMinutes() * 60 + this.date.getSeconds()
    const timeInput = document.getElementById('time')

    timeInput.value = time
    timeInput.oninput = () => {
      time = +timeInput.value
      this.date.setHours(Math.floor(time / 60 / 60))
      this.date.setMinutes(Math.floor(time / 60) % 60)
      this.date.setSeconds(time % 60)
      map.triggerRepaint()
    }

    // The 'building' layer in the mapbox-streets vector source contains building-height
    // data from OpenStreetMap.
    const _this = this
    map.on('style.load', function () {
      // stats

      map.getContainer().appendChild(_this.stats.dom)

      // _this.animate()

      // const mapCenter = map.getCenter()
      // const center = [mapCenter.lng, mapCenter.lat]
      p.innerHTML = 'Local time: ' + _this.dateToTimezone(_this.date, point.timezone).toLocaleTimeString()

      map.addSource(_this.mapConfig.names.fillExtrusion, {
        // GeoJSON Data source used in vector tiles, documented at
        // https://gist.github.com/ryanbaumann/a7d970386ce59d11c16278b90dde094d
        type: 'geojson',
        generateId: true, // _This ensures that all features have unique IDs and allow selecting them
        data: './geojson/statue-of-liberty.geojson'
      })

      // map.addLayer(_this.createExtrusionLayer())

      map.addLayer({
        id: _this.mapConfig.names.customLayer,
        type: 'custom',
        renderingMode: '3d',
        onAdd: function (map, mbxContext) {
          // Creative Commons License attribution: Liberty statue model by https://sketchfab.com/hellolucy2
          // from https://sketchfab.com/3d-models/ellis-island-3cd765a23c5c4c7087acd00624d30590

          const options = {
            obj: 'eiffel.glb',
            type: 'gltf',
            scale: _this.mapConfig.NYC.scale,
            units: 'meters',
            rotation: { x: 90, y: 0, z: 0 }, // default rotation
            anchor: 'center'
          }

          window.tb.loadObj(options, function (model) {
            model.setCoords(_this.mapConfig.NYC.origin)
            model.setRotation({ x: 0, y: 0, z: -147 })
            model.addTooltip('Statue of Liberty', true)
            model.castShadow = true
            window.tb.add(model)
          })

          // Creative Commons License attribution:  Eiffel Tower model by https://www.cgtrader.com/lefabshop
          // https://www.cgtrader.com/items/108594/download-page
          // options = {
          //   obj: 'residential_complex.glb',
          //   type: 'gltf',
          //   scale: _this.mapConfig.PAR.scale,
          //   units: 'meters',
          //   rotation: { x: 90, y: 0, z: 0 } // default rotation
          // }

          // window.tb.loadObj(options, function (model) {
          //   model.setCoords(_this.mapConfig.PAR.origin)
          //   model.setRotation({ x: 0, y: 0, z: 45.7 })
          //   model.addTooltip('Eiffel Tower', true)
          //   model.castShadow = true
          //   window.tb.add(model)
          // })
        },

        render: function (gl, matrix) {
          window.tb.setSunlight(_this.date, point.center)
          // _this.hour.innerHTML = 'Local date/time: ' + _this.dateToTimezone(_this.date, point.timezone).toLocaleTimeString()
          window.tb.update()
        }
      })
    })
  },
  methods: {
    // animate () {
    //   window.requestAnimationFrame(this.animate)
    //   this.stats.update()
    // },
    createExtrusionLayer () {
      const layer = {
        id: this.mapConfig.names.fillExtrusionLayer,
        source: this.mapConfig.names.fillExtrusion,
        filter: ['==', 'extrude', 'true'],
        type: 'fill-extrusion',
        minzoom: this.minZoom,
        paint: {
          'fill-extrusion-color': ['case', ['boolean', ['feature-state', 'select'], false], 'lightgreen', ['boolean', ['feature-state', 'hover'], false], 'lightblue', '#C7B299'],
          // use an 'interpolate' expression to add a smooth transition effect to the
          // buildings as the user zooms in
          'fill-extrusion-height': [
            'interpolate',
            ['linear'],
            ['zoom'],
            this.minZoom,
            0,
            this.minZoom + 0.05,
            ['get', 'height']
          ],
          'fill-extrusion-base': [
            'interpolate',
            ['linear'],
            ['zoom'],
            this.minZoom,
            0,
            this.minZoom + 0.05,
            ['get', 'min_height']
          ],
          'fill-extrusion-opacity': 1
        }
      }
      return layer
    },
    dateToTimezone (date = new Date(), timezone) {
      const tzTime = date.toLocaleString('en-US', { timeZone: timezone })
      return new Date(tzTime)
    },
    reset () {
      this.date = new Date()
      this.seconds = 0
    }
  }
}
</script>

<style scoped>
#map {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}
#time {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 80px;
  margin-left: auto;
  margin-right: auto;
  min-width: 90%;
}
</style>
