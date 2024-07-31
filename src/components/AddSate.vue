<template>
    <div id="cesiumContainer">
        <div class="buttons"><button id="Satellites" @click="toState()">Satellites</button>
            <button id="Vehicle" @click="toVehicle()">Vehicle</button>
            <button id="reset" @click="reset()">reset</button>
        </div>
    </div>

</template>
<script setup>
import * as Cesium from 'cesium'
import { onMounted, ref } from 'vue';

const viewer = ref(null);
onMounted(() => {
    Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
    viewer.value = new Cesium.Viewer("cesiumContainer", {
        shouldAnimate: true,
    });
})
const toState = async () => {
    console.log("Loading Vehicle CZML...");
    await viewer.value.dataSources.add(

        Cesium.CzmlDataSource.load("/SampleData/simple.czml")
    );
    viewer.value.camera.flyHome(0);
}

const toVehicle = async () => {
    await viewer.value.dataSources.add(
        Cesium.CzmlDataSource.load("/SampleData/Vehicle.czml")
    );
    viewer.value.scene.camera.setView({
        destination: Cesium.Cartesian3.fromDegrees(-116.52, 35.02, 95000),
        orientation: {
            heading: 6,
        },
    });
}
const reset = () => {
    viewer.value.dataSources.removeAll();
}
</script>

<style>
#cesiumContainer {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.buttons {
    position: absolute;
    display: inline-block;
    top: 0;
    left: calc(50% - 100px);
    z-index: 1;
}
</style>