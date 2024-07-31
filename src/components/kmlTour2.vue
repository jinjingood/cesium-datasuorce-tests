<template>
    <div id="cesiumContainer">
        <div class="buttons">
            <button @click="change(0)">Play</button>
            <button @click="change(1)">Terminate</button>
        </div>
    </div>

</template>

<script setup>
const emits = defineEmits(['change']);//传给父组件必写
const change = (val) => {
    emits('change', val)
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