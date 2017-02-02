<template>
    <div id="app">
        <GoogleMap v-on:mapclick="onMapClick" v-on:paths="onPathsChanged" :points="points" class="map"></GoogleMap>
        <div class="sidebar">
            <div class="export">
                <button v-on:click="onPreview()">Preview SVG</button>
                <button v-on:click="onExport()">Export SVG</button>
            </div>
            <div class="points-list">
                <ul>
                    <li v-for="(point, index) in points">
                        <span class="text" :title="point.latLng">{{ index + 1 }}. {{ point.latLng }}</span>
                        <span class="delete" v-on:click="onDeletePoint(point)">X</span>
                    </li>
                </ul>
            </div>
            <div class="preview">
                <svg id="svg-preview" xmlns="http://www.w3.org/2000/svg" xmlns:svg="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="240px" height="120px" viewBox="0 0 640 480">
                    <title>A test SVG file</title>
                    <defs>
                        <svg:style>
                            text {
                                font-family: serif;
                                stroke: none;
                                fill: red;
                            }
                            .underline {
                                stroke: blue;
                                stroke-width: 1;
                                fill: none;
                                marker-mid: url(#arrow);
                            }
                        </svg:style>
                        <marker id="arrow"
                                viewBox="-3 -3 6 6" orient="auto"
                                markerUnits="strokeWidth"
                                refX="0" refY="0"
                                markerWidth="6" markerHeight="6">
                            <path d="M0,0 -3,-3 3,0 -3,3 Z"/>
                        </marker>
                    </defs>
                    <text id="changeable-text" text-anchor="middle" font-size="40" x="320" y="240"></text>
                    <path class="underline" d="M10,250 h310 310"/>
                </svg>
            </div>
        </div>
    </div>
</template>

<script>
import _ from 'lodash';

import Map from './components/Map.vue';
import Point from './models/Point.js';

export default {
    name: 'app',
    components: {
        GoogleMap: Map
    },
    data() {
        return {
            message: 'andWelcome to Your Vue.js App',
            points: [],
            paths: []
        }
    },
    watch: {
        paths: function(paths) {
            if (!paths.length) return;
            var svg = document.getElementById("svg-preview");
            var elements = svg.getElementsByTagName("path");
            for (var index = elements.length - 1; index >= 0; --index) {
                elements[index].parentNode.removeChild(elements[index]);
            }
            var minX = 180, maxX = -180, minY = 90, maxY = -90;
            minX = Math.min(minX, paths[0][0].lng());
            maxX = Math.max(maxX, paths[0][0].lng());
            minY = Math.min(minY, paths[0][0].lat());
            maxY = Math.max(maxY, paths[0][0].lat());
            var d = "M " + paths[0][0].lng() + ' ' + (-paths[0][0].lat());
            for (var i = 0; i < paths.length; ++i) {
                for (var j = 0; j < paths[i].length; ++j) {
                    if (i == 0 && j == 0) continue;
                    minX = Math.min(minX, paths[i][j].lng());
                    maxX = Math.max(maxX, paths[i][j].lng());
                    minY = Math.min(minY, paths[i][j].lat());
                    maxY = Math.max(maxY, paths[i][j].lat());
                    d += ' L ' + paths[i][j].lng() + ' ' + (-paths[i][j].lat());
                }
                var path = document.createElementNS(svg.namespaceURI, "path");
                // path.setAttributeNS(null, "id", "pathIdD");
                path.setAttributeNS(null, "d", d);
                path.setAttributeNS(null, "stroke", "black");
                path.setAttributeNS(null, "stroke-width", 0.00005);
                path.setAttributeNS(null, "opacity", 1);
                path.setAttributeNS(null, "fill", "none");
                svg.appendChild(path);
            }
            console.log(minX, maxX, minY, maxY);
            svg.setAttribute("viewBox", minX + ' ' + (-maxY) + ' ' + (maxX - minX) + ' ' + (maxY - minY)); 
        }
    },
    methods: {
        onExport() {
            var svg = document.getElementById("svg-preview");
            var serializer = new XMLSerializer();
            var svg_blob = new Blob([serializer.serializeToString(svg)], {'type': "image/svg+xml"});
            var reader = new window.FileReader();
            reader.readAsText(svg_blob); 
            reader.onloadend = function() {
                var base64data = reader.result;

                var element = document.createElement('a');
                element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(base64data));
                element.setAttribute('download', 'export.svg');

                element.style.display = 'none';
                document.body.appendChild(element);

                element.click();

                document.body.removeChild(element);
            }
        },
        onPreview() {
            var svg = document.getElementById("svg-preview").cloneNode(true);
            svg.setAttribute('width', '100%');
            svg.setAttribute('height', '100%');
            var serializer = new XMLSerializer();
            var svg_blob = new Blob([serializer.serializeToString(svg)], {'type': "image/svg+xml"});
            var url = URL.createObjectURL(svg_blob);
            var svg_win = window.open(url, "svg_win");
        },
        onPathsChanged(paths) {
            this.paths = paths;
        },
        onMapClick(latLng) {
            var point = new Point(latLng);
            this.points.push(point);
        },
        onDeletePoint(point) {
            this.points = _.without(this.points, point);
        }
    }
}
</script>

<style lang="scss">
@import "normalize";

#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
}

.map {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 250px;
}

.sidebar {
    display: flex;
    flex-direction: column;
    position: absolute;
    top: 0;
    bottom: 0;
    right: 0;
    width: 250px;

    .export {
        flex: 0;
        margin: 5px 0;
        text-align: center;
    }

    .points-list {
        flex: 1;
        overflow: auto;

        ul {
            list-style-type: none;
            margin: 0;
            padding: 0;

            li {
                display: flex;
                padding: 5px 10px;

                &:hover {
                    background-color: rgba(0, 0, 0, 0.05);
                }

                &:not(:last-child) {
                    border-bottom: 1px dotted gray;
                }

                .text {
                    flex: 1;
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                }

                .delete {
                    cursor: pointer;
                    display: none;
                    flex: 0;

                    &:hover {
                        color: red;
                    }
                }

                &:hover .delete {
                    display: block;
                }
            }
        }
    }

    .preview {
        border-top: 1px solid gray;
        height: 120px;
    }
}
</style>
