<template>
    <div id="cesiumContainer">
        <div class="buttons">
            <button @click="change(1)">Track with Waypoints</button>
            <button @click="change(2)">Route</button>
            <button @click="change(3)">Waypoints</button>
            <button @click="change(4)">Multiple Tracks with Waypoints</button>
            <button @click="change(5)">Symbology Options</button>
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
let promise1 = null
let promise2 = null
let promise3 = null
let promise4 = null
let promise5 = null

onMounted(async () => {
    cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
    viewer = new cesium.Viewer('cesiumContainer', {
        infoBox: false,
        terrain: cesium.Terrain.fromWorldTerrain(),
    })
    initial()//先默认把5种gpx数据都加载上
    viewer.zoomTo(promise1);//然后让视图默认定位在某个gpx数据上
    //写法B
    // viewer.zoomTo(dataSources[0]);
})

const initial = () => {
    let pinBuilder = new cesium.PinBuilder();
    promise1 = viewer.dataSources.add(cesium.GpxDataSource.load(lamina, { clampToGround: true, }))
    promise2 = viewer.dataSources.add(cesium.GpxDataSource.load(route, { clampToGround: true, }))
    promise3 = viewer.dataSources.add(cesium.GpxDataSource.load(wpt, { clampToGround: true, }))
    promise4 = viewer.dataSources.add(cesium.GpxDataSource.load(complexTrk, { clampToGround: true, }))
    promise5 = viewer.dataSources.add(cesium.GpxDataSource.load(lamina, {
        clampToGround: true, trackColor: cesium.Color.YELLOW,
        waypointImage: pinBuilder.fromMakiIconId(
            "bicycle",
            cesium.Color.BLUE,
            48
        ),
    }))
    //promise1、5有移动动画，所以要加下面2个
    viewer.clock.clockRange = cesium.ClockRange.UNBOUNDED;
    viewer.clock.clockStep = cesium.ClockStep.SYSTEM_CLOCK;
}

const change = async (value) => {
    // await viewer.dataSources.removeAll();//切换按钮时，先移除全部gpx数据
    // initial()//然后再加载一遍这些数据
    //上面这2步好像不用写，不影响效果
    if (value === 1) {
        viewer.zoomTo(promise1)
    } else if (value === 2) {
        viewer.zoomTo(promise2)
    } else if (value === 3) {
        viewer.zoomTo(promise3)
    } else if (value === 4) {
        viewer.zoomTo(promise4)
    } else {
        viewer.zoomTo(promise5)
    }
    //这里没写viewer.clock.clockRange和viewer.clock.clockStep，所以切换时，动画是继续播放，不是从头播放
};

//写法B
// let dataSources = []
// const initial = () => {
//     let pinBuilder = new cesium.PinBuilder();
//     promise1 = cesium.GpxDataSource.load(lamina, { clampToGround: true, })
//     promise2 = cesium.GpxDataSource.load(route, { clampToGround: true, })
//     promise3 = cesium.GpxDataSource.load(wpt, { clampToGround: true, })
//     promise4 = cesium.GpxDataSource.load(complexTrk, { clampToGround: true, })
//     promise5 = cesium.GpxDataSource.load(lamina, {
//         clampToGround: true,
//         trackColor: cesium.Color.YELLOW,
//         waypointImage: pinBuilder.fromMakiIconId(
//             "bicycle",
//             cesium.Color.BLUE,
//             48
//         ),
//     })
//     dataSources.push(promise1, promise2, promise3, promise4, promise5)
//     Promise.all(dataSources.map(dataSource => viewer.dataSources.add(dataSource)));
// }

// const change = async (value) => {
//     await viewer.dataSources.removeAll();
//     await viewer.dataSources.add(dataSources[value]);
//     viewer.zoomTo(dataSources[value])
// };


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