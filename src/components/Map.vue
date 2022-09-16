<template>
    <div id="map">
    </div>

    <div>
        <p> {{this.lat}}</p>
        <p> {{this.long}}</p>
    </div>
</template>
<script>

import Feature from "ol/Feature";
import Map from "ol/Map";
import View from "ol/View";
import { Circle, Point } from "ol/geom";
import { OSM, Vector as VectorSource } from "ol/source";
import { Icon, Style } from "ol/style";
import { Tile as TileLayer, Vector as VectorLayer } from "ol/layer";

export default {
    name: 'Map',
    data: function () {
        return {
            errorStr: null,
            options: {
                enableHighAccuracy: true,
                timeout: 2000,
                maximumAge: 0
            },
            lat: 0,
            long: 0,
        }
    },
    methods: {
        async onLoad() {
            return new Promise((resolve) => {
                if (!(navigator.geolocation)) {
                    this.errorStr = 'Geolocation is not available.';
                    return;
                }
                navigator.geolocation.watchPosition(pos => {
                    resolve(pos)
                }, err => {
                    this.errorStr = err.message
                    if (this.errorStr == 'User denied Geolocation') {
                        window.alert('Favor ligar a localização')
                    }
                },
                    this.options)
            })
        },
    },
    async mounted() {

        let data = await this.onLoad()

        this.lat = data.coords.latitude
        this.long = data.coords.longitude

        //onsole.log(`More or less ${this.options.enableHighAccuracy} meters.`)

        const circleFeature = new Feature({
            geometry: new Circle([data.coords.longitude, data.coords.latitude], 0.001)
        });

        const pointFeature = new Feature({
            geometry: new Point([data.coords.longitude, data.coords.latitude])
        });

        pointFeature.setStyle(
            new Style({
                image: new Icon({
                    anchor: [0.5, 1],
                    anchorXUnits: "fraction",
                    //anchorXUnits: "pixels",
                    scale: 0.05,
                    src: "https://cdn-icons-png.flaticon.com/512/684/684908.png"
                })
            })
        );

        circleFeature.setStyle(
            new Style({
                renderer(coordinates, state) {
                    const [[x, y], [x1, y1]] = coordinates;
                    const ctx = state.context;
                    const dx = x1 - x;
                    const dy = y1 - y;
                    const radius = Math.sqrt(dx * dx + dy * dy);
                    const innerRadius = 0;
                    const outerRadius = radius * 1.1;
                    const gradient = ctx.createRadialGradient(
                        x,
                        y,
                        innerRadius,
                        x,
                        y,
                        outerRadius
                    );
                    gradient.addColorStop(0, "rgba(30,144,255,0)");
                    gradient.addColorStop(0.5, "rgba(30,144,255,0.1)");
                    gradient.addColorStop(1, "rgba(30,144,255,0.3)");
                    ctx.beginPath();
                    ctx.arc(x, y, radius, 0, 2 * Math.PI, true);
                    ctx.fillStyle = gradient;
                    ctx.fill();
                }
            })
        );

        this.map = new Map({
            controls: [],
            layers: [
                controls: [],
                new TileLayer({
                    source: new OSM({
                        attributions: null,
                        controls: []
                        
                    }),
                         attributions: null,
                    }),
                    visible: true
                }),
                new VectorLayer({
                    source: new VectorSource({
                        features: [pointFeature, circleFeature]
                    })
                })
            ],
            target: "map",
            view: new View({
                center: [data.coords.longitude, data.coords.latitude],
                zoom: 17,
                projection: "EPSG:4326"
            })
        })
    },

}
</script>
