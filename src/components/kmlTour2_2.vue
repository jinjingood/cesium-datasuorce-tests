<template>
    <div id="cesiumContainer">
        <div class="buttons">
            <button @click="change(0)">Play</button>
            <button @click="change(1)">Terminate</button>
            <button @click="change(3)">reset</button>
        </div>
    </div>


</template>

<script setup>
const emits = defineEmits(['change']);//传给父组件必写
const change = (val) => {
    emits('change', val)
    console.log("kml子组件", val);
}
</script>

<style>
.buttons {
    position: absolute;
    left: 25%;
    z-index: 1;
    display: flex;
    justify-content: space-between;
}
</style>