<!-- 还有报错，得研究 -->
<template>
    <div id="cesiumContainer">
        <div class="box">
            <button class="buttons" @click="change">Reset demo</button>
            <div class="statusDisplay">
                <p v-for="(it, index) in msg" :key="it">{{ it }}&nbsp;&nbsp;<span>{{ status[index] }}</span></p>
            </div>
            <div class="fuelDisplay"> Fuel:{{ textContent }}gal</div>
        </div>

    </div>
</template>

<script setup>
import * as Cesium from 'cesium'
import { onMounted, ref, reactive } from 'vue'
import { MultipartVehicle_part1 } from "../components/czml/MultipartVehicle_part1.js"
import { MultipartVehicle_part2 } from "../components/czml/MultipartVehicle_part2.js"
import { MultipartVehicle_part3 } from "../components/czml/MultipartVehicle_part3.js"

Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI4MmUwNWJjOC0xNmZmLTRhOGUtYTEzNi1iYTM3Mjk2YzU3NmYiLCJpZCI6MTYzNzE0LCJpYXQiOjE2OTM0NjkzMDJ9.k80jNxQNVAl9ijau5Swcp1eGqUHY3mTqLScj5gr6bC4'

let viewer
let vehicleEntity
let dataSource
let textContent = ref(null)//注意一定要用响应式ref，否则div那里拿不到它的变化
let msg = ['MultipartVehicle_part1', 'MultipartVehicle_part2', 'MultipartVehicle_part3'];
let status = ref([])//同上注意ref
const preloadTimeInSeconds = 100;

let partsToLoad = reactive([
    {
        url: MultipartVehicle_part1,
        lable: 'MultipartVehicle_part1',
        range: [0, 1500],
        requested: false,
        loaded: false,
    },
    {
        url: MultipartVehicle_part2,
        lable: 'MultipartVehicle_part2',
        range: [1500, 3000],
        requested: false,
        loaded: false,
    },
    {
        url: MultipartVehicle_part3,
        lable: 'MultipartVehicle_part3',
        range: [3000, 4500],
        requested: false,
        loaded: false,
    },
]);

let timeOffset
// let clock = new Cesium.Viewer.clock

onMounted(() => {
    viewer = new Cesium.Viewer("cesiumContainer", {
        infoBox: false,
        shouldAnimate: true,
    })
    dataSource = new Cesium.CzmlDataSource();
    viewer.dataSources.add(dataSource);
    processPart(partsToLoad[0]);
    time()
})

const time = () => {
    // 下面是官网写的，但是我发现放omounted或者变成函数（const time）放在omounted外面都不影响效果，注释掉也不影响，为什么？？
    viewer.clock.onTick.addEventListener(function (clock) {
        // This example uses time offsets from the start to identify which parts need loading.
        timeOffset = Cesium.JulianDate.secondsDifference(
            clock.currentTime,
            clock.startTime
        );

        // Filter the list of parts to just the ones that need loading right now.
        // Then, process each part that needs loading.
        partsToLoad
            .filter(function (part) {
                return (
                    !part.requested &&
                    timeOffset >= part.range[0] - preloadTimeInSeconds &&
                    timeOffset <= part.range[1]
                );
            })
            .forEach(function (part) {
                processPart(part);
            });

        if (vehicleEntity) {
            const fuel = vehicleEntity.properties.fuel_remaining.getValue(
                clock.currentTime
            );
            if (Cesium.defined(fuel)) {
                textContent.value = fuel.toFixed(2)
                //   fuelDisplay.textContent = `Fuel: ${fuel.toFixed(2)} gal`;
            }
        }
    });
}



const updateStatusDisplay = () => {
    let statu = ref('')
    partsToLoad.forEach((part) => {
        if (part.loaded) {
            statu.value = "加载完成";
        } else if (part.requested) {
            statu.value = "加载中...";
        } else {
            statu.value = "还未加载";
        }
        status.value.push(statu.value)
        console.log('msg.value', msg.value);
    });

}

const processPart = (part) => {
    part.requested = true;
    updateStatusDisplay();
    dataSource.process(part.url).then(() => {
        part.loaded = true;
        updateStatusDisplay();
        if (!viewer.trackedEntity) {
            viewer.trackedEntity = vehicleEntity = dataSource.entities.getById(
                "Vehicle"
            );
        }
    });
}

const change = async () => {
    viewer.clock.currentTime = viewer.clock.startTime;
    viewer.clock.shouldAnimate = true;
    partsToLoad.forEach((part) => {
        part.requested = false;
        part.loaded = false;
    });
    dataSource.entities.removeAll();
    processPart(partsToLoad[0]);
    console.log('partsToLoad', partsToLoad);
}

</script>

<style>
.box {
    position: absolute;
    top: 20px;
    left: 20px;
    z-index: 1;
}

.statusDisplay,
.fuelDisplay {
    background: rgba(42, 42, 42, 0.7);
    padding: 5px 10px;
    margin-top: 5px;
}
</style>