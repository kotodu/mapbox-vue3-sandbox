<template>
    <img alt="Vue logo" src="./assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js + TypeScript App" />
    <div id="map"></div>
</template>

<script lang="ts">
import { Map, Style } from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { onMounted, reactive } from "vue";
import { Options, Vue } from "vue-class-component";
import HelloWorld from "./components/HelloWorld.vue";

@Options({
    components: {
        HelloWorld,
    },
    mounted() {
        const mapstyle = reactive<Style>({
            version: 8,
            sources: {
                OSM: {
                    type: "raster",
                    tiles: ["http://tile.openstreetmap.org/{z}/{x}/{y}.png"],
                    tileSize: 256,
                    attribution:
                        '<a href="http://osm.org/copyright">© OpenStreetMap contributors</a>',
                },
            },
            layers: [
                {
                    id: "OSM",
                    type: "raster",
                    source: "OSM",
                    minzoom: 0,
                    maxzoom: 18,
                },
            ],
        });
        const token =
            "pk.eyJ1Ijoia290b2R1IiwiYSI6ImNranMxaTlwdjFoM3EyeWszdGZqMXJsdjEifQ.R34mEmruOGlukBv73g1CDA";
        const map = new Map({
            accessToken: token,
            container: "map",
            style: mapstyle,
            center: [139.767, 35.681],
            zoom: 11,
        });
    },
})
export default class App extends Vue {}
</script>

<style lang="scss">
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
#map {
    height: 400px;
}
</style>
