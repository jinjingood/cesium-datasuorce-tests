<template>
  <div id="cesiumContainer">
    <kmlTour2_2 @change="playStop" v-if="status === 0" />
    <CustomDatasuorce v-if="status === 1" />
  </div>

</template>
<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue';
import kmlTour2_2 from "./components/kmlTour2_2.vue";
import eiffel from './components/kmldata/eiffel.kml'
import CustomDatasuorce from './components/CustomDatasuorce.vue'

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
let viewer = null;
let tour = null
let status = 1

onMounted(() => {
  if (status === 0) {
    getKML()
  } else if (status === 1) {
    getCustomData()
  }
})

const getKML = () => {
  viewer = new Cesium.Viewer("cesiumContainer", { infoBox: false, animation: false })
  const options = { camera: viewer.scene.camera, canvas: viewer.scene.canvas, }
  const load = viewer.dataSources.add(Cesium.KmlDataSource.load(eiffel, options))//这里的url只能在这个父组件引入使用，子组件传地址过来没用，load不到kml地址上的数据，tour会null
  load.then((dataSource) => {
    tour = dataSource.kmlTours[0]
    console.log('then.tour', tour);//后执行这个(说明then是等待了一会较晚完成的）
  })
  console.log('tour', tour);//先执行这个
}

const playStop = (value) => {
  reset()
  if (value === 0) {
    tour.play(viewer.cesiumWidget)
  } else if (value === 1) {
    tour.stop(viewer.cesiumWidget)
  } else {
    reset()
  }
  viewer.clock.clockRange = Cesium.ClockRange.UNBOUNDED
  viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK
}

const getCustomData = () => {
  viewer = new Cesium.Viewer("cesiumContainer", { infoBox: false })
}
</script>

<style scoped>
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