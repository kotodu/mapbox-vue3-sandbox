<template>
    <div id="map"></div>
</template>

<script lang="ts">
import { Map, Style } from "mapbox-gl";
import MapboxDraw from "@mapbox/mapbox-gl-draw";
import "mapbox-gl/dist/mapbox-gl.css";

// v8はドキュメントなしなのでgithubを直接見ること
import { Options, Vue } from "vue-class-component";

const mapstyle: Style = {
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
};
const token = process.env.VUE_APP_TOKEN;

const draw = new MapboxDraw({
    displayControlsDefault: false,
    controls: {
        // @ts-ignore
        line_string: true,
        trash: true,
    },
    styles: [
        // Set the line style for the user-input coordinates
        {
            id: "gl-draw-line",
            type: "line",
            filter: [
                "all",
                ["==", "$type", "LineString"],
                ["!=", "mode", "static"],
            ],
            layout: {
                "line-cap": "round",
                "line-join": "round",
            },
            paint: {
                "line-color": "#438EE4",
                "line-dasharray": [0.2, 2],
                "line-width": 4,
                "line-opacity": 0.7,
            },
        },
        // Style the vertex point halos
        {
            id: "gl-draw-polygon-and-line-vertex-halo-active",
            type: "circle",
            filter: [
                "all",
                ["==", "meta", "vertex"],
                ["==", "$type", "Point"],
                ["!=", "mode", "static"],
            ],
            paint: {
                "circle-radius": 12,
                "circle-color": "#FFF",
            },
        },
        // Style the vertex points
        {
            id: "gl-draw-polygon-and-line-vertex-active",
            type: "circle",
            filter: [
                "all",
                ["==", "meta", "vertex"],
                ["==", "$type", "Point"],
                ["!=", "mode", "static"],
            ],
            paint: {
                "circle-radius": 8,
                "circle-color": "#438EE4",
            },
        },
    ],
});
// https://docs.mapbox.com/help/tutorials/get-started-map-matching-api/

const getMatch = async (
    coordinates: string,
    radius: number[],
    profile: string
) => {
    const radiuses = radius.join(";");
    const query =
        "https://api.mapbox.com/matching/v5/mapbox/" +
        profile +
        "/" +
        coordinates +
        "?geometries=geojson&radiuses=" +
        radiuses +
        "&steps=true&access_token=" +
        process.env.VUE_APP_TOKEN;
    const response = await fetch(query);
    const data = await response.json();
    return data.matchings[0].geometry;
};

@Options({
    mounted() {
        const map = new Map({
            accessToken: token,
            container: "map",
            style: mapstyle,
            center: [139.767, 35.681],
            zoom: 11,
        });
        const updateRoute = async () => {
            const profile = "driving";
            const data = draw.getAll();
            const lastFeature = data.features.length - 1;
            const point = data.features[lastFeature].geometry;
            const coords =
                point.type === "GeometryCollection"
                    ? new Array()
                    : point.coordinates;
            const newCoords = coords.join(";");
            const radius = coords.map(coord => {
                return 25;
            });
            const geometry = await getMatch(newCoords, radius, profile);
            if (map.getSource("route")) {
                map.removeLayer("route");
                map.removeSource("route");
            } else {
                map.addLayer({
                    id: "route",
                    type: "line",
                    source: {
                        type: "geojson",
                        data: {
                            type: "Feature",
                            properties: {},
                            // @ts-ignore
                            geometry: geometry,
                        },
                    },
                    layout: {
                        "line-join": "round",
                        "line-cap": "round",
                    },
                    paint: {
                        "line-color": "#03AA46",
                        "line-width": 8,
                        "line-opacity": 0.8,
                    },
                });
            }
        };
        map.addControl(draw);
        map.on("draw.create", updateRoute);
        map.on("draw.update", updateRoute);
    },
})
export default class App extends Vue {}
</script>

<style lang="scss">
body {
    margin: 0;
}
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin: 0;
}
#map {
    height: 100vh;
    width: 100vw;
}
</style>
