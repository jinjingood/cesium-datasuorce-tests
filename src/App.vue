<template>
  <!-- <cesium-Viewer /> -->
  <!-- <add-Sate /> -->
  <!-- <data-Ordering /> -->
  <!-- <GPX /> -->
  <!-- <GPX2 /> -->
  <!-- <ThreeDLayer /> -->
  <!-- <kmlTour/> -->
  <!-- <MultiPart /> -->
  <!-- <Simplestyles /> -->
  <!-- <IntegratedMesh /> -->
  <TopoJson />
  <!-- <CustomDatasuorce /> -->

  <!-- 注意：这里外面没有 <div id="cesiumContainer">，而kmlTour2组件里有，类名在这能看见，所以new Cesium.Viewer还是能识别到"cesiumContainer" -->
  <!-- <kmlTour2 @change="playStop" /> -->

  <!-- 下面这些是有问题或完全没成功的 -->
  <!-- <SimpleKML /> -->
</template>
<script setup>
// import CesiumViewer from './components/CesiumViewer.vue
// import AddSate from './components/AddSate.vue'
// import DataOrdering from './components/DataOrdering.vue'
// import GPX from './components/GPX.vue'//这里的写法有点问题，在切换到symbology options时，看不到蓝色图标，被覆盖了
// import GPX2 from './components/GPX2.vue'//这里改进了GPX的问题，按需加载了gpt数据，而不是始终全部加载，那样红色图标会覆盖在蓝色图标上
import TopoJson from "./components/TopoJSON.vue";
// import ThreeDLayer from "./components/ThreeDLayer.vue";
// import kmlTour from "./components/kmlTour.vue";
// import CustomDatasuorce from "./components/CustomDatasuorce.vue";
// import SimpleKML from './components/SimpleKML.vue'
// import Simplestyles from './components/Simplestyles.vue'
// import IntegratedMesh from './components/IntegratedMesh.vue'
// import MultiPart from "./components/MultiPart.vue";

// 这是给kmlTour2用的
// import * as Cesium from 'cesium'
// import { onMounted } from 'vue';
// import kmlTour2 from "./components/kmlTour2.vue";
// import eiffel from './components/kmldata/eiffel.kml'

// Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
// let viewer = null;
// let tour = null
// let options = {//单独设置个变量用来存储new Cesium.Viewer中的{}，这样可以动态的修改窗口
//   infoBox: false
// }
// options.animation = false
// options.timeline = false

// onMounted(async () => {
//   viewer = new Cesium.Viewer("cesiumContainer", { infoBox: false })
//   const options = { camera: viewer.scene.camera, canvas: viewer.scene.canvas, }
//   const load = viewer.dataSources.add(Cesium.KmlDataSource.load(eiffel, options))
//   load.then((dataSource) => {
//     tour = dataSource.kmlTours[0]
//   })
//   console.log('tour', tour);
// })

// const playStop = (value) => {
//   console.log("父组件", value);
//   if (value === 0) {
//     tour.play(viewer.cesiumWidget)
//   } else {
//     tour.stop(viewer.cesiumWidget)
//   }
//   viewer.clock.clockRange = Cesium.ClockRange.UNBOUNDED
//   viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK
// }

</script>

<style scoped>
#cesiumContainer {
  position: absolute;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100vw;
}
</style>