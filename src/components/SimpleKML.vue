<template>
    <div id="cesiumContainer">
        <div class="buttons">
            <button @click="change(1)">1-KML - Global Science Facilities</button>
            <button @click="change(2)">2-KMZ with embedded data - GDP per capita</button>
            <button @click="change(3)">3-gx KML extensions - Bike Ride</button>
        </div>
    </div>
</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue'
import facilities from "../components/kmldata/facilities.kml"
import gdpPerCapita2008 from "../components/kmldata/gdpPerCapita2008.kml"//kml/kmz格式都行
import bikeride from "../components/kmldata/bikeride.kml"
// import simplestyles from '../components/jsdata/simplestyles.geojson'

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'

let viewer = null
let data1
let data2
let data3
// let dataSource


onMounted(async () => {
    viewer = new Cesium.Viewer('cesiumContainer', {
        infoBox: false,
        terrain: Cesium.Terrain.fromWorldTerrain(),
    })
    const options = {
        camera: viewer.scene.camera,
        canvas: viewer.scene.canvas,
        screenOverlayContainer: viewer.container,
    };
    data1 = Cesium.KmlDataSource.load(facilities, options)
    data2 = Cesium.KmlDataSource.load(gdpPerCapita2008, options)
    data3 = Cesium.KmlDataSource.load(bikeride, options)
    data3.then((dataSource) => {
        viewer.clock.shouldAnimate = false;
        const rider = dataSource.entities.getById("tour");
        viewer.flyTo(rider).then(() => {
            viewer.trackedEntity = rider;
            viewer.selectedEntity = viewer.trackedEntity;
            viewer.clock.multiplier = 30;
            viewer.clock.shouldAnimate = true;
        })
    })
    // dataSource = await Cesium.GeoJsonDataSource.load(simplestyles);
    // viewer.dataSources.add(dataSource);
    // viewer.zoomTo(dataSource);
})


const change = (value) => {
    reset()
    if (value === 1) {
        viewer.camera.flyHome(0);
        viewer.dataSources.add(data1)
    } else if (value === 2) {
        viewer.camera.flyHome(0);
        viewer.dataSources.add(data2)
    } else {
        viewer.dataSources.add(data3)
    }
    viewer.clock.clockRange = Cesium.ClockRange.UNBOUNDED;
    viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK;
}

const reset = () => {
    viewer.dataSources.removeAll();
    //     viewer.clock.clockRange = Cesium.ClockRange.UNBOUNDED;
    //     viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK; 
};
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
    top: 0;
    z-index: 1;
}
</style>