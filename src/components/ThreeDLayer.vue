<template>
    <div id="cesiumContainer"></div>
</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue';

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'

let viewer = null;
let model = 'ThreeDLyaer'
let options = {//单独设置个变量用来存储new Cesium.Viewer中的{}，这样可以动态的修改窗口
    // infoBox: false//这里不能禁用infoBox，因为会阻挡点击地图展示信息弹窗
    animation: false
}

options.timeline = false

const tours = {
    "San Francisco":
        "https://tiles.arcgis.com/tiles/z2tnIkrLQ2BRzr6P/arcgis/rest/services/SanFrancisco_3DObjects_1_7/SceneServer/layers/0",
};

onMounted(async () => {
    if (model === 'ThreeDLyaer') {
        options.terrain = Cesium.Terrain.fromWorldTerrain(),
            viewer = new Cesium.Viewer("cesiumContainer", options)
    }
    try {
        const geoidService = await Cesium.ArcGISTiledElevationTerrainProvider.fromUrl(
            "https://tiles.arcgis.com/tiles/z2tnIkrLQ2BRzr6P/arcgis/rest/services/EGM2008/ImageServer"
        );
        const i3sOptions = {
            geoidTiledTerrainProvider: geoidService, // pass the geoid service
        };
        const i3sProvider = await Cesium.I3SDataProvider.fromUrl(
            tours["San Francisco"],
            i3sOptions
        );
        viewer.scene.primitives.add(i3sProvider);
        const center = Cesium.Rectangle.center(i3sProvider.extent);
        center.height = 5000.0;
        viewer.camera.setView({
            destination: Cesium.Ellipsoid.WGS84.cartographicToCartesian(center),
        });
    } catch (error) {
        console.log(
            `There was an error creating the I3S Data Provider: ${error}`
        );
    }
    
    const selectedEntity = new Cesium.Entity();

    const onLeftClick = (movement) => {
        const pickedFeature = viewer.scene.pick(movement.position);
        if (!Cesium.defined(pickedFeature)) {
            return;
        }

        const pickedPosition = viewer.scene.pickPosition(movement.position);

        if (
            Cesium.defined(pickedFeature.content) &&
            Cesium.defined(pickedFeature.content.tile.i3sNode)
        ) {
            const i3sNode = pickedFeature.content.tile.i3sNode;
            if (pickedPosition) {
                i3sNode.loadFields().then(function () {
                    let description = "No attributes";
                    let name;

                    const fields = i3sNode.getFieldsForPickedPosition(
                        pickedPosition
                    );
                    if (Object.keys(fields).length > 0) {
                        description =
                            '<table class="cesium-infoBox-defaultTable"><tbody>';
                        for (const fieldName in fields) {
                            if (i3sNode.fields.hasOwnProperty(fieldName)) {
                                description += `<tr><th>${fieldName}</th><td>`;
                                description += `${fields[fieldName]}</td></tr>`;
                                if (!Cesium.defined(name) && isNameProperty(fieldName)) {
                                    name = fields[fieldName];
                                }
                            }
                        }
                        description += `</tbody></table>`;
                    }
                    if (!Cesium.defined(name)) {
                        name = "unknown";
                    }
                    selectedEntity.name = name;
                    selectedEntity.description = description;
                    viewer.selectedEntity = selectedEntity;
                });
            }
        }

    }

    viewer.screenSpaceEventHandler.setInputAction(onLeftClick, Cesium.ScreenSpaceEventType.LEFT_CLICK)

    // viewer.screenSpaceEventHandler.setInputAction(function onLeftClick(
    //     movement
    // ) {
    //     // Pick a new feature
    //     const pickedFeature = viewer.scene.pick(movement.position);
    //     if (!Cesium.defined(pickedFeature)) {
    //         return;
    //     }

    //     const pickedPosition = viewer.scene.pickPosition(movement.position);

    //     if (
    //         Cesium.defined(pickedFeature.content) &&
    //         Cesium.defined(pickedFeature.content.tile.i3sNode)
    //     ) {
    //         const i3sNode = pickedFeature.content.tile.i3sNode;
    //         if (pickedPosition) {
    //             i3sNode.loadFields().then(function () {
    //                 let description = "No attributes";
    //                 let name;

    //                 const fields = i3sNode.getFieldsForPickedPosition(
    //                     pickedPosition
    //                 );
    //                 if (Object.keys(fields).length > 0) {
    //                     description =
    //                         '<table class="cesium-infoBox-defaultTable"><tbody>';
    //                     for (const fieldName in fields) {
    //                         if (i3sNode.fields.hasOwnProperty(fieldName)) {
    //                             description += `<tr><th>${fieldName}</th><td>`;
    //                             description += `${fields[fieldName]}</td></tr>`;
    //                             if (!Cesium.defined(name) && isNameProperty(fieldName)) {
    //                                 name = fields[fieldName];
    //                             }
    //                         }
    //                     }
    //                     description += `</tbody></table>`;
    //                 }
    //                 if (!Cesium.defined(name)) {
    //                     name = "unknown";
    //                 }
    //                 selectedEntity.name = name;
    //                 selectedEntity.description = description;
    //                 viewer.selectedEntity = selectedEntity;
    //             });
    //         }
    //     }
    // },
    //     Cesium.ScreenSpaceEventType.LEFT_CLICK);

    const isNameProperty = (propertyName) => {
        const name = propertyName.toLowerCase();
        if (
            name.localeCompare("name") === 0 ||
            name.localeCompare("objname") === 0
        ) {
            return true;
        }
        return false;
    }
})
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