<!-- 关键错误点：
1-不要随便改构造函数function WebGLGlobeDataSource为箭头函数，作用非常不一样，不记得了就去百度
2-不要用官网写的loadUrl，咱这用不着，因为json文件已经存入本地了，不是从服务器获取的（确切原理不是特别懂） -->
<template>
    <div id="cesiumContainer">
        <div class="buttons">
            <button v-for='it in seriesName' :key="it" @click="change(it)">{{ it }}</button>
        </div>
    </div>

</template>

<script setup>
import * as Cesium from 'cesium';
import { onMounted, reactive } from 'vue';
import population909500 from '../components/jsondata/population909500.json'

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'
let viewer
let seriesName = reactive([])

//‘_’是标识私用变量的意思
//下面的Object.defineProperties只能给构造函数加，普通对象是无法添加原型方法的，所以这里必须写成构造的，这是构造了一个“类”
function WebGLGlobeDataSource(name) {
    this._name = name;
    this._changed = new Cesium.Event();
    this._error = new Cesium.Event();
    this._isLoading = false;
    this._loading = new Cesium.Event();
    this._entityCollection = new Cesium.EntityCollection();
    this._seriesNames = [];
    this._seriesToDisplay = undefined;
    this._heightScale = 10000000;
    this._entityCluster = new Cesium.EntityCluster();
}

//上面用构造函数的形式，是为了将方法（如：164行的load、237行的_setLoading）放到这个函数的原型上（即prototype内），
//这样在后续拿这个函数创建实例时，这些属性就能直接跟所有实例共享。而不会在创建实例时又拷贝一份方法了，这可以减少内存，提升性能。
Object.defineProperties(WebGLGlobeDataSource.prototype, {
    name: {
        get: function () {
            return this._name;
        },
    },
    clock: {
        value: undefined,
        writable: false,
    },
    entities: {
        get: function () {
            return this._entityCollection;
        },
    },
    isLoading: {
        get: function () {
            return this._isLoading;
        },
    },
    changedEvent: {
        get: function () {
            return this._changed;
        },
    },
    errorEvent: {
        get: function () {
            return this._error;
        },
    },
    loadingEvent: {
        get: function () {
            return this._loading;
        },
    },
    seriesNames: {
        get: function () {
            return this._seriesNames;
        },
    },
    seriesToDisplay: {
        get: function () {
            return this._seriesToDisplay;
        },
        set: function (value) {
            this._seriesToDisplay = value;
            const collection = this._entityCollection;
            const entities = collection.values;
            collection.suspendEvents();
            for (let i = 0; i < entities.length; i++) {
                const entity = entities[i];
                entity.show = value === entity.seriesName;//表示把value === entity.seriesName结果（true/false）赋值给 entity.show
            }
            collection.resumeEvents();
        },
    },
    heightScale: {
        get: function () {
            return this._heightScale;
        },
        set: function (value) {
            if (value <= 0) {
                throw new Cesium.DeveloperError("value must be greater than 0");
            }
            this._heightScale = value;
        },
    },
    show: {
        get: function () {
            return this._entityCollection;
        },
        set: function (value) {
            this._entityCollection = value;
        },
    },
    clustering: {
        get: function () {
            return this._entityCluster;
        },
        set: function (value) {
            if (!Cesium.defined(value)) {
                throw new Cesium.DeveloperError("value must be defined.");
            }
            this._entityCluster = value;
        },
    },

})

//不需要loadUrl方法，这是用来从服务器请求json文件的，我这里永远请求不到，系统只会指向index.html，导致报错
// WebGLGlobeDataSource.prototype.loadUrl = async function (url) {//不能改为箭头函数，否则this指向不正常
//     if (!Cesium.defined(url)) {
//         throw new Cesium.DeveloperError("url is required.");
//     }
//     // console.log('url', url);

//     const name = Cesium.getFilenameFromUri(url);
//     console.log("name", name);

//     if (this._name !== name) {
//         this._name = name;
//         this._changed.raiseEvent(this);
//     }
//     // const that = this;
//     // console.log('that',that);

//     return Cesium.Resource.fetchJson(url)
//         .then((response) => {
//             if (!response.ok) {
//                 throw new Error(`HTTP error! status: ${response.status}`);
//             }
//             return response.json();
//         })
//         .then((json) => {//改为箭头函数，让that、this指向准确
//             console.log('url', url);
//             console.log('json', json);
//             return this.load(json, url);
//         })
//         .catch((error) => {
//             this._setLoading(false);
//             this._error.raiseEvent(this, error);
//             return Promise.reject(error);
//         });
// };

WebGLGlobeDataSource.prototype.load = function (data) {//这里必须用function构造函数，否则里面的this会指向不明
    if (!Cesium.defined(data)) {
        throw new Cesium.DeveloperError("data is required.");
    }
    this._setLoading(true);
    this._seriesNames.length = 0;
    this._seriesToDisplay = undefined;

    const heightScale = this.heightScale;
    const entities = this._entityCollection;

    console.log('entities',entities);
    
    entities.suspendEvents();
    entities.removeAll();

    for (let x = 0; x < data.length; x++) {
        const series = data[x];
        const seriesName = series[0];
        const coordinates = series[1];
        this._seriesNames.push(seriesName);

        const show = x === 0;
        if (show) {
            this._seriesToDisplay = seriesName;
        }

        for (let i = 0; i < coordinates.length; i += 3) {
            const latitude = coordinates[i];
            const longitude = coordinates[i + 1];
            const height = coordinates[i + 2];

            if (height === 0) {
                continue;
            }

            const color = Cesium.Color.fromHsl(0.6 - height * 0.5, 1.0, 0.5);
            const surfacePosition = Cesium.Cartesian3.fromDegrees(
                longitude,
                latitude,
                0
            );
            const heightPosition = Cesium.Cartesian3.fromDegrees(
                longitude,
                latitude,
                height * heightScale
            );

            const polyline = new Cesium.PolylineGraphics();
            polyline.material = new Cesium.ColorMaterialProperty(color);
            polyline.width = new Cesium.ConstantProperty(2);
            polyline.arcType = new Cesium.ConstantProperty(
                Cesium.ArcType.NONE
            );
            polyline.positions = new Cesium.ConstantProperty([
                surfacePosition,
                heightPosition,
            ]);

            const entity = new Cesium.Entity({
                id: `${seriesName} index ${i.toString()}`,
                show: show,
                polyline: polyline,
                seriesName: seriesName, //Custom property to indicate series name
            });

            entities.add(entity);
        }
    }

    entities.resumeEvents();
    this._changed.raiseEvent(this);
    this._setLoading(false);
};

WebGLGlobeDataSource.prototype._setLoading = function (isLoading) {
    if (this._isLoading !== isLoading) {
        this._isLoading = isLoading;
        this._loading.raiseEvent(this, isLoading);
    }
};

let dataSource
onMounted(() => {
    dataSource = new WebGLGlobeDataSource();
    dataSource.load(population909500)//这里去调用160写的load，把population909500的地址传过去了，也就是data

    for (let i = 0; i < dataSource.seriesNames.length; i++) {
        seriesName.push(dataSource.seriesNames[i]);
    }

    viewer = new Cesium.Viewer("cesiumContainer", {
        infoBox: false,
        animation: false,
        timeline: false,
    });
    viewer.clock.shouldAnimate = false;
    viewer.dataSources.add(dataSource);
    //下面打印出来是WebGLGlobeDataSource全部属性24、164行
    // console.log('dataSource', dataSource);

    //下面打印出来是aa是24行构造函数那堆代码
    // const aa = WebGLGlobeDataSource
    // console.log('aa', aa);
});

const change = (seriesName) => {
    dataSource.seriesToDisplay = seriesName;
}
</script>

<style scoped>
#cesiumContainer {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.buttons {
    position: absolute;
    display: inline-block;
    top: 0;
    left: calc(50% - 100px);
    z-index: 1;
}
</style>
