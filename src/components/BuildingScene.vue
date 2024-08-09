<template>
    <div id="cesiumContainer">
        <div class="toolbar" id="toolbar"></div>
    </div>
</template>
<script setup>
import * as Cesium from 'cesium';
import { onMounted, } from 'vue';

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
let viewer

const tours = {
    "Turanga Library":
        "https://tiles.arcgis.com/tiles/cFEFS0EWrhfDeVw9/arcgis/rest/services/Turanga_Library/SceneServer",
};

onMounted(async () => {
    viewer = new Cesium.Viewer("cesiumContainer", {
        infoBox: false,
        terrain: new Cesium.Terrain(
            Cesium.ArcGISTiledElevationTerrainProvider.fromUrl(
                "https://elevation3d.arcgis.com/arcgis/rest/services/WorldElevation3D/Terrain3D/ImageServer"
            )
        ),
        animation: false,
        timeline: false,
        orderIndependentTranslucency: false,//用于控制是否启用顺序无关透明度（OIT）渲染技术
        //场景中有大量的半透明对象：当你有许多半透明对象，比如云层、烟雾、玻璃等，并且它们彼此交叠时，OIT 可以自动正确地处理它们的渲染顺序。
        //提高渲染质量：使用 OIT 可以减少半透明对象渲染中的视觉伪影，比如不正确的透明度叠加问题。
        //简化开发工作：开发者不需要手动对半透明对象进行排序，Cesium 会自动处理。
        //但它也会增加 GPU 的计算负担，特别是在场景中有大量半透明对象时。因此，在低性能设备上或者需要优化性能时，可能需要考虑禁用 OIT。
        msaaSamples: 4,
    });

    try {
        const i3sOptions = {
            adjustMaterialAlphaMode: true, // force the alpha mode to be set for transparent geometry
            showFeatures: true, // creates 3D object for each feature and allows to apply attributes filter
            applySymbology: true, // applies outlines based on the I3S layer renderer details
            calculateNormals: true, // generates flat normals if they are missing in I3S buffers
        };
        const i3sProvider = await Cesium.I3SDataProvider.fromUrl(
            tours["Turanga Library"],
            i3sOptions
        );
        Cesium.I3SBuildingSceneLayerExplorer("toolbar", i3sProvider);
        viewer.scene.primitives.add(i3sProvider);
        const center = Cesium.Rectangle.center(i3sProvider.extent);
        center.height = 300.0;
        let bslLayer = i3sProvider.data;
        if (Cesium.defined(bslLayer.layers) && bslLayer.layers.length > 0) {
            bslLayer = bslLayer.layers[0];
        }
        if (
            Cesium.defined(bslLayer.fullExtent) &&
            Cesium.defined(bslLayer.fullExtent.zmax)
        ) {
            center.height += bslLayer.fullExtent.zmax;
        }
        const destination = Cesium.Ellipsoid.WGS84.cartographicToCartesian(
            center
        );
        viewer.camera.setView({
            destination: destination,//初始化摄像机画面
        });
        viewer.homeButton.viewModel._command = Cesium.createCommand(
            // function () {//匿名函数箭头函数都可以
            () => {
                viewer.camera.flyTo({
                    destination: destination,//按cesium的home键以动画形式回归初始画面
                })
                // }
            }
        );
    } catch (error) {
        console.error(
            `There was an error creating the I3S Data Provider: ${error}`
        );
    }

    const selected = {
        feature: undefined,
        originalColor: new Cesium.Color(),
    };
    const selectedEntity = new Cesium.Entity();
    // Show metadata in the InfoBox.
    viewer.screenSpaceEventHandler.setInputAction(function onLeftClick(
        movement
    ) {
        const selectedFeature = selected.feature;

        // Pick a new feature
        const pickedFeature = viewer.scene.pick(movement.position);
        if (selectedFeature === pickedFeature) {
            return;
        }

        // If a feature was previously selected, undo the highlight
        if (Cesium.defined(selected.feature)) {
            selected.feature.color = selected.originalColor;
            selected.feature = undefined;
        }

        if (
            !Cesium.defined(pickedFeature) ||
            !Cesium.defined(pickedFeature.content) ||
            !Cesium.defined(pickedFeature.content.tile.i3sNode)
        ) {
            viewer.selectedEntity = undefined;
            return;
        }

        // Highlight newly selected feature
        selected.feature = pickedFeature;
        Cesium.Color.clone(pickedFeature.color, selected.originalColor);
        pickedFeature.color = Cesium.Color.BLUE;

        const i3sNode = pickedFeature.content.tile.i3sNode;
        i3sNode.loadFields().then(function () {
            let description = "No attributes";
            let name;

            const fields = i3sNode.getFieldsForFeature(pickedFeature.featureId);
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
    },
        Cesium.ScreenSpaceEventType.LEFT_CLICK);

    function isNameProperty(propertyName) {
        const name = propertyName.toLowerCase();
        if (
            name.localeCompare("name") === 0 ||
            name.localeCompare("objname") === 0 ||
            name.localeCompare("category") === 0
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
    width: 100%;
    height: 100%;
}

.toolbar {
    position: absolute;
    top: 20px;
    left: 20px;
    background: rgba(42, 42, 42, 0.8);
    padding: 10px;
    border-radius: 4px;
    max-height: 100%;
    overflow-y: auto;
    z-index: 1;
}
</style>