<template>
    <div id="app">
        <jx3box-map :map-id="8" :datas="datas"></jx3box-map>
    </div>
</template>

<script>
import Jx3boxMap from "./components/Map.vue";

export default {
    name: "App",
    components: {
        Jx3boxMap,
    },
    data: () => ({
        datas: [
            {
                x: 9655,
                y: 13231,
                title: "测试",
                content: "测试内容11<br />22",
            },
        ],
    }),
};
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

#app {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100vw;
    height: 100vh;
}
</style>
