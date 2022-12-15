<template>
    <div>
        <div>
            {{ users[0].latitude }},{{ users[0].longitude }}
        </div>
        <div id="map-div"></div>
    </div>
</template>
<style>
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
                    longitude: 0
                }
            ],
            map: null
        }
    },
    mounted() {
        if (navigator.geolocation) {
            navigator.geolocation.watchPosition(this.updatePosition);
        }
    },
    methods: {
        updatePosition(position) {
            this.users[0].latitude = position.coords.latitude;
            this.users[0].longitude = position.coords.longitude;

            this.map = tt.map({
                key: process.env.VUE_APP_TOMTOM_API_KEY,
                container: 'map-div',
                center: {
                    lng: this.users[0].longitude, 
                    lat: this.users[0].latitude
                },
                zoom: 12
            });
        }
    }
}
</script>