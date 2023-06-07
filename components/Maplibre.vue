<template>
  <section>
    <div>
      <div id="map" class="map" style="width: 100vw; height: 100vh" />
    </div>
    <pre id="info" />
  </section>
</template>

<script>
import maplibregl from 'maplibre-gl'
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
    const a = {
      tilejson: '2.2.0',
      name: 'korea3',
      id: 'korea3',
      scheme: 'xyz',
      tiles: [
        'http://192.168.0.52:8080/data/korea3/{z}/{x}/{y}.png'
      ],
      minzoom: 0,
      maxzoom: 18,
      center: [126.875, 37.55081252406616],
      zoom: 12,
      version: 8,
      sources: {
        korea3: {
          type: 'raster',
          tiles: ['http://192.168.0.52:8080/data/korea3/{z}/{x}/{y}.png'],
          tileSize: 256,
          minzoom: 6,
          maxzoom: 18,
          bounds: [112.5, 27.059125784374068, 146.25, 48.922499263758255]
        }
      },
      layers: [{
        id: 'simple-tiles',
        type: 'raster',
        source: 'korea3',
        minzoom: 6,
        maxzoom: 18
      }]
    }
    this.map = new maplibregl.Map({
      container: 'map', // container id
      style: a
    })
  },
  methods: {

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
