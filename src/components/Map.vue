<template>
   <div id = "map">
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
    data(){
        return {
            lat: 0,
            long: 0
        }
    },
     computed:{
        lat(){
            return this.lat
        },
        long(){
            return this.long
        }
     }
     ,
    methods:{
        onLoad:function(){
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(this.showPosition);
                console.log("--- Rodou o get Location ---");
            } else {
                console.log("---  Não rodou ---");
            }
        },
        showPosition: function(position) {
            this.lat = position.coords.latitude;
                console.log(this.lat)
                //localStorage.setItem("lat", this.lat);
            this.long = position.coords.longitude;
                console.log(this.long)
            }
    },
    mounted() {
        this.onLoad()
        console.log(parseInt(this.lat) + " vv")
        //this.showPosition()

        const circleFeature = new Feature({
            geometry: new Circle([ this.long,  this.lat], 0.001)
        });

        const pointFeature = new Feature({
            geometry: new Point([-49.374100250480424, -28.679954764955344])
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
            layers: [
                new TileLayer({
                source: new OSM(),
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
                center: [-49.374100250480424, -28.679954764955344],
                zoom: 17,
                projection: "EPSG:4326"
            })
        })
    },

}
  </script>