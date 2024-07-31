<template>
    <div id="cesiumContainer">
        <button class="button" @click="swap()">swap</button>
    </div>

</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted, ref } from 'vue';
import { czml1, czml2 } from '../components/jsdata/czmlMap.js'

const viewer = null//不要用ref，否则会报错：allow-scripts
let promise1 = null
let promise2 = null

onMounted(() => {
    Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
    viewer = new Cesium.Viewer('cesiumContainer', {
    })
    inital()
})
const inital = () => {
    promise1 = viewer.dataSources.add(Cesium.CzmlDataSource.load(czml1));
    promise2 = viewer.dataSources.add(Cesium.CzmlDataSource.load(czml2));
}

const swap = () => {
    Promise.all([promise1, promise2]).then(function (results) {
        const ds1 = results[0];
        const ds2 = results[1];
        if (viewer.dataSources.indexOf(ds1) === 0) {
            viewer.dataSources.raise(ds1);
        } else {
            viewer.dataSources.lower(ds1);
            // viewer.dataSources.raise(ds2);
        }
    });
}

//swap可以这样写
// const swap = () => {
//     Promise.all([promise1, promise2]).then((results) => {//results拿到的是一个包含promise1和promise2的数组
//         const ds1 = results[0]
//         // const ds2 = results[1]
//         if (viewer.dataSources.indexOf(ds1) === 0) {
//             viewer.dataSources.raise(ds1);
//         } else {
//             viewer.dataSources.lower(ds1);
//         }

//     })
// }

</script>

<style>
#cesiumContainer {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.button {
    position: absolute;
    z-index: 1;

}
</style>./jsdata/czmlMap.js