<template>
    <div id="cesiumContainer">
        <div class="buttons">
            <button @click="change(0)">Default styling</button>
            <button @click="change(1)">Basic styling</button>
            <button @click="change(2)">Custom styling</button>
        </div>
    </div>
</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted } from 'vue';
import ne_10m_us_states from '../components/jsondata/ne_10m_us_states.json'

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'

let viewer = null;
let promise = null
let number = 0

onMounted(() => {
    viewer = new Cesium.Viewer("cesiumContainer", {
        infoBox: false,
        shouldAnimate: true,
    });
    loadGeo()
    viewer.camera.lookAt(
        Cesium.Cartesian3.fromDegrees(-98.0, 40.0),
        new Cesium.Cartesian3(0.0, -4790000.0, 3930000.0)
    );
    viewer.camera.lookAtTransform(Cesium.Matrix4.IDENTITY);
})

const load = (url, options) => {
    return viewer.dataSources.add(Cesium.GeoJsonDataSource.load(url, options))
}

const loadGeo = () => {
    if (number === 0) {
        promise = load(ne_10m_us_states)
    } else if (number === 1) {
        promise = load(ne_10m_us_states, {
            stroke: Cesium.Color.HOTPINK,
            fill: Cesium.Color.PINK.withAlpha(0.5),
            strokeWidth: 3,
        })
    } else {
        Cesium.Math.setRandomNumberSeed(0);
        promise = load(ne_10m_us_states)
        promise.then((result) => {
            console.log(result);
            //注意这里promise没用Promise().then,而是直接在自定义的变量promise后面.then
            const entities = result.entities.values;//注意看这里是result的entities，通常是dataSource.entities,而这里变成了then里的result。
            //也就是说dataSource并不是官方带什么默认属性的方法
            const colorHash = {};
            for (let i = 0; i < entities.length; i++) {
                const entity = entities[i];
                const name = entity.name;
                let color = colorHash[name];
                if (!color) {
                    color = Cesium.Color.fromRandom({
                        alpha: 1.0,
                    });
                    colorHash[name] = color;
                }

                entity.polygon.material = color;
                entity.polygon.outline = false;

                entity.polygon.extrudedHeight =
                    entity.properties.Population / 50.0;
            }
            // viewer.zoomTo(promise)//会让camera失效
        })
            .catch((error) => {
                window.alert(error);
            });
    }
}

const change = (value) => {
    viewer.dataSources.removeAll();
    number = value
    loadGeo()
}
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