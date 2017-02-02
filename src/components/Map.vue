<template>
    <div></div>
</template>

<script>
export default {
    name: 'Map',
    props: {
        points: Array
    },
    data: function() {
        return {
            directionsDisplay: new google.maps.DirectionsRenderer(),
            directionsService: new google.maps.DirectionsService(),
            markers: [],
            map: null,
            mapOptions: {
                zoom: 16,
                mapTypeId: google.maps.MapTypeId.ROADMAP,
                // Chicago
                center: new google.maps.LatLng(41.855, -87.65)
            },

            paths: [],
            pathsCache: {},
            polylines: []
        }
    },
    computed: {
        pairs() {
            if (!this.points.length) return [];
            return _.reduce(this.points, (pairs, point) => {
                if (pairs.length) pairs[pairs.length - 1].push(point);
                pairs.push([point]);
                return pairs;
            }, []).slice(0, -1);
        }
    },
    watch: {
        paths: function(paths) {
            this.$emit('paths', paths);
        },
        pairs: function(pairs) {
            this.clearPaths();
            _.each(pairs, pair => {
                var cached = this.pathsCache[pair[0].latLng + pair[1].latLng];
                if (cached) this.addPath(cached);
                else this.getDirections(pair[0].latLng, pair[1].latLng, path => {
                    this.pathsCache[pair[0].latLng + pair[1].latLng] = path;
                    this.addPath(path);
                });
            });
        },
        points: function(points) {
            var count = points.length;
            for (var i = points.length; i < this.markers.length; ++i) {
                this.markers[i].setMap(null);
            }
            this.markers.length = count;
            for (var i = 0; i < count; ++i) {
                if (!this.markers[i]) this.markers[i] = new google.maps.Marker();
                this.markers[i].setOptions({
                    label: '' + (i + 1),
                    map: this.map,
                    position: points[i].latLng
                });
            }
        }
    },
    mounted() {
        this.map = new google.maps.Map(this.$el, this.mapOptions);
        this.directionsDisplay.setMap(this.map);
        google.maps.event.addListener(this.map, 'click', (e) => {
            this.onClick(e.latLng);
        });
    },
    methods: {
        addPath(path) {
            var polyline = new google.maps.Polyline({
                path: path,
                strokeColor: '#FF0000',
                strokeOpacity: 0.8,
                strokeWeight: 3,
                fillColor: '#FF0000',
                fillOpacity: 0.35,
                map: this.map
            });
            polyline.setMap(this.map);
            this.polylines.push(polyline);
            this.paths.push(path);
        },
        clearPaths() {
            _.each(this.polylines, polyline => {
                polyline.setMap(null);
            });
            this.polylines = [];
            this.paths = [];
        },
        getDirections(latLngFrom, latLngTo, callback) {
            var request = {
                origin: latLngFrom,
                destination: latLngTo,
                travelMode: google.maps.DirectionsTravelMode.DRIVING
            };
            this.directionsService.route(request, (response, status) => {
                if (status == google.maps.DirectionsStatus.OK) {
                    var path = google.maps.geometry.encoding.decodePath(response.routes[0].overview_polyline);
                    callback(path);
                }
            });
        },
        onClick(latLng) {
            this.$emit('mapclick', latLng);
            // if (!this.last) {
            //     this.last = latLng;
            //     return;
            // }
            // this.marker.setMap(null);

            // var request = {
            //     origin: this.last,
            //     destination: latLng,
            //     travelMode: google.maps.DirectionsTravelMode.DRIVING
            // };
            // this.last = latLng;
            // this.directionsService.route(request, (response, status) => {
            //     console.log(response);
            //     if (status == google.maps.DirectionsStatus.OK) {
            //         var point = response.routes[0].legs[0];
            //         this.marker.setOptions({
            //             map: this.map,
            //             position: point.start_location
            //         });
            //         this.map.setCenter(point.start_location);
            //         alert(response.routes[0].summary + '\n' + point.start_location.toString());

            //         var path = google.maps.geometry.encoding.decodePath(response.routes[0].overview_polyline);
            //         console.log(path);

            //         var polyline = new google.maps.Polyline({
            //             path: path,
            //             strokeColor: '#FF0000',
            //             strokeOpacity: 0.8,
            //             strokeWeight: 2,
            //             fillColor: '#FF0000',
            //             fillOpacity: 0.35,
            //             map: this.map
            //         });
            //         polyline.setMap(this.map);
            //     }
            // });
        }
    }
}
</script>
