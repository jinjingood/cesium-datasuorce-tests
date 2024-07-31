<template>
  <div id="cesiumContainer">
    <button id="toggle-building">Toggle new building</button>
  </div>

</template>


<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue'

// defineProps({
//   msg: String,
// })
onMounted(async () => {
  Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
  const viewer = new Cesium.Viewer('cesiumContainer', {
    terrain: Cesium.Terrain.fromWorldTerrain(),
  })
  const buildingsTileset = await Cesium.createOsmBuildingsAsync();
  viewer.scene.primitives.add(buildingsTileset);

  const addBuildingGeoJSON = async () => {
    const geoJSONURL = await Cesium.IonResource.fromAssetId(2674965);
    const geoJSON = await Cesium.GeoJsonDataSource.load(geoJSONURL, { clampToGround: false });// 官方教程是true，但是会提示无法显示轮廓要设高度为0，gpt教的解决方案之一：改为false禁用地形钳制
    const dataSource = await viewer.dataSources.add(geoJSON);
    for (const entity of dataSource.entities.values) {
      entity.polygon.classificationType = Cesium.ClassificationType.TERRAIN;
    }
  }
  addBuildingGeoJSON()

  buildingsTileset.style = new Cesium.Cesium3DTileStyle({
    // Create a style rule to control each building's "show" property.
    show: {
      conditions: [
        // 这是在按照id隐藏cesium自己3d建筑物（点击建筑物就能找到对应的elementid）
        ['${elementId} === 332469316', false],
        ['${elementId} === 332469317', false],
        ['${elementId} === 235368665', false],
        ['${elementId} === 530288180', false],
        ['${elementId} === 530288179', false],
        ['${elementId} === 34008043', false],
        // If a building does not have one of these elementIds, set `show = true`.
        [true, true]
      ]
    },
    //如果建筑物没有color属性，就用#ff0000
    color: "Boolean(${feature['cesium#color']}) ? color(${feature['cesium#color']}) : color('#ff0000')"
  });
  const newBuildingTileset = await Cesium.Cesium3DTileset.fromIonAssetId(2674965);
  viewer.scene.primitives.add(newBuildingTileset);
  viewer.flyTo(newBuildingTileset);
  document.querySelector('#toggle-building').onclick = function () {
    newBuildingTileset.show = !newBuildingTileset.show;
    buildingsTileset.show = !buildingsTileset.show//可以隐藏cesium自己的建筑群
  }

})

</script>



<style scoped>
#cesiumContainer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

#toggle-building {
  z-index: 1;
  position: fixed;
  top: 5px;
  left: 5px;
}
</style>
