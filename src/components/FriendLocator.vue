<template>
    <div>
        <div>
            {{ users[0].latitude }},{{ users[0].longitude }}
        </div>
        <div id="map-div"></div>
    </div>
</template>
<style>
    @import 'https://api.tomtom.com/maps-sdk-for-web/cdn/6.x/6.21.3/maps/maps.css';
    body, html { margin: 0; padding: 0; }
    #map-div { width: 100vw; height: 100vh; }
</style>
<script>
import tt from '@tomtom-international/web-sdk-maps'

export default {
    data() {
        return {
            users: [
                {
                    latitude: 0,
                    longitude: 0,
                    marker: null
                }
            ],
            map: null
        }
    },
    mounted() {
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(this.initializeMap);
            navigator.geolocation.watchPosition(this.updatePosition);
        }
    },
    methods: {
        initializeMap(position) {
            this.map = tt.map({
                key: process.env.VUE_APP_TOMTOM_API_KEY,
                container: 'map-div',
                center: {
                    lng: position.coords.longitude, 
                    lat: position.coords.latitude
                },
                zoom: 12
            });
            this.users[0].marker = new tt.Marker().setLngLat([position.coords.longitude, position.coords.latitude]).addTo(this.map);
        },
        updatePosition(position) {
            this.users[0].latitude = position.coords.latitude;
            this.users[0].longitude = position.coords.longitude;
            this.users[0].marker.setLngLat([position.coords.longitude, position.coords.latitude]);
        }
    }
}
</script>