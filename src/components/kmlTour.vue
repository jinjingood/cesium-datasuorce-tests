<template>

        <div class="buttons">
            <button @click="change(0)">Play</button>
            <button @click="change(1)">Terminate</button>
        </div>


</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue'
import eiffel from '../components/kmldata/eiffel.kml'

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
let viewer = null;

let tour = null

onMounted(() => {
    viewer = new Cesium.Viewer('cesiumContainer', {
        infoBox: false
    })
    // const options = { infoBox: false, camera: viewer.scene.camera, canvas: viewer.scene.canvas, }
    const load = viewer.dataSources.add(Cesium.KmlDataSource.load(eiffel, options))
    load.then((dataSource) => {
        tour = dataSource.kmlTours[0]
        //写官网这些交互就不正常了，虽然没有报错
        // tour.tourStart.addEvetListener = () => {
        //     console.log("Start tour");
        // }
        // tour.tourEnd.addEventListener = (terminated) => {
        //     console.log(`${terminated ? "Terminate" : "End"} tour`);
        // };
        // tour.entryStart.addEventListener = (entry) => {
        //     console.log(`Play ${entry.type} (${entry.duration})`);
        // };
        // tour.entryEnd.addEventListener = (entry, terminated) => {
        //     console.log(`${terminated ? "Terminate" : "End"} ${entry.type}`);
        // };
    })
})

const play = () => {
    tour.play(viewer.cesiumWidget)
}

const Terminate = () => {
    tour.stop(viewer.cesiumWidget)
}

const change = (value) => {
    reset()
    if (value === 0) {
        play()
    } else {
        Terminate()
    }
}

const reset = () => {
    viewer.dataSources.removeAll();
    viewer.clock.clockRange = Cesium.ClockRange.UNBOUNDED
    viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK
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