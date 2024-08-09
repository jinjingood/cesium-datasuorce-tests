<template>
    <div id="cesiumContainer"></div>
</template>

<script setup>
import * as cesium from 'cesium'
import { onMounted } from 'vue'
import simplestyles from '../components/jsondata/simplestyles.json'

let viewer
onMounted(() => {
    viewer = new cesium.Viewer('cesiumContainer', {
        infoBox: false
    })
    const dataSource = viewer.dataSources.add(cesium.GeoJsonDataSource.load(simplestyles))
    viewer.zoomTo(dataSource)
})

</script>