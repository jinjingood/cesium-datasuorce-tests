<template>
    <div id="cesiumContainer">
        <div class="buttons">
            <button @click="change(0)">Track with Waypoints</button>
            <button @click="change(1)">Route</button>
            <button @click="change(2)">Waypoints</button>
            <button @click="change(3)">Multiple Tracks with Waypoints</button>
            <button @click="change(4)">Symbology Options</button>
        </div>
    </div>
</template>

<script setup>
import * as cesium from 'cesium'
import { onMounted } from 'vue';
import lamina from '../components/gpxdata/lamina.gpx';//注意lamina别加{}
import route from '../components/gpxdata/route.gpx';
import wpt from '../components/gpxdata/wpt.gpx';
import complexTrk from '../components/gpxdata/complexTrk.gpx';

let viewer = null
let promise = null
let number = 0
let dataMap = [lamina, route, wpt, complexTrk]


onMounted(async () => {
    cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
    viewer = new cesium.Viewer('cesiumContainer', {
        infoBox: false,
        terrain: cesium.Terrain.fromWorldTerrain(),
    })
    await loadGPX()
})

const loadGPX = async () => {
    viewer.dataSources.removeAll()
    let pinBuilder = new cesium.PinBuilder();
    if (number !== 4) {
        promise = viewer.dataSources.add(cesium.GpxDataSource.load(dataMap[number], { clampToGround: true, }))
    } else {
        promise = viewer.dataSources.add(cesium.GpxDataSource.load(lamina, {
            clampToGround: true, trackColor: cesium.Color.YELLOW,
            waypointImage: pinBuilder.fromMakiIconId(
                "bicycle",
                cesium.Color.BLUE,
                48
            ),
        }))
    }
    viewer.zoomTo(promise)
    viewer.clock.clockRange = cesium.ClockRange.UNBOUNDED;
    viewer.clock.clockStep = cesium.ClockStep.SYSTEM_CLOCK;
};

const change = (value) => {
    number = value
    loadGPX()
}

</script>

<style>
#cesiumContainer {
    position: absolute;
    top: 0;
    left: 0;
    height: 100vh;
    width: 100vw;
}

.buttons {
    position: absolute;
    left: 25%;
    z-index: 1;
    display: flex;
    justify-content: space-between;
}
</style>