<template>
    <div id="cesiumContainer"></div>
</template>

<script setup>
import * as cesium from 'cesium'
import { onMounted } from 'vue'

cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'

let viewer
onMounted(async () => {
    viewer = new cesium.Viewer('cesiumContainer', {
        infoBox: false,
        terrain: cesium.Terrain.fromWorldTerrain(),
        timeline: false,
        animation: false,
    })
    // 抑制地形数据以避免与 IntegratedMesh 图层几何体发生冲突
    viewer.scene.globe.depthTestAgainstTerrain = false;
    const tours = {
        Frankfurt:
            "https://tiles.arcgis.com/tiles/z2tnIkrLQ2BRzr6P/arcgis/rest/services/Frankfurt2017_vi3s_18/SceneServer/layers/0",
    };
    try {

        const geoidService = await cesium.ArcGISTiledElevationTerrainProvider.fromUrl(
            "https://tiles.arcgis.com/tiles/z2tnIkrLQ2BRzr6P/arcgis/rest/services/EGM2008/ImageServer"
        );

        // Create i3s options to pass optional parameters useful for debugging and visualizing
        const i3sOptions = {
            geoidTiledTerrainProvider: geoidService, // pass the geoid service
        };
        // Create I3S data provider
        const i3sProvider = await cesium.I3SDataProvider.fromUrl(
            tours.Frankfurt,
            i3sOptions
        );
        // Add the i3s layer provider as a primitive data type
        viewer.scene.primitives.add(i3sProvider);

        // Center camera on I3S once it's loaded
        const center = cesium.Rectangle.center(i3sProvider.extent);
        center.height = 5000.0;
        viewer.camera.setView({
            destination: cesium.Ellipsoid.WGS84.cartographicToCartesian(center),
        });
    } catch (error) {
        console.log(
            `There was an error creating the I3S Data Provider: ${error}`)
    }
})






</script>