<template>
    <div>
        <div>
            {{ user.location.latitude }},{{ user.location.longitude }}
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
    props: ['connection', 'participants', 'userId'],
    data() {
        return {
            markers: {},
            map: null
        }
    },
    computed: {
        user() {
            return this.participants.find(p => p.uuid === this.userId);
        }
    },
    mounted() {
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(this.initializeMap);
            navigator.geolocation.watchPosition(this.updatePosition);
        }
        else {
            const dummyLocation = {
                coords: {
                    latitude: 22.634834,
                    longitude: 88.768484
                }
            }
            this.initializeMap(dummyLocation);
            setInterval(() => {
                dummyLocation.coords.latitude++;
                dummyLocation.coords.longitude++;
                this.updatePosition(dummyLocation);
            }, 2000);
        }
    },
    updated() {
        this.updateMarkers()
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
            // this.users[0].marker = new tt.Marker().setLngLat([position.coords.longitude, position.coords.latitude]).addTo(this.map);
        },
        updatePosition(position) {
            // this.users[0].latitude = position.coords.latitude;
            // this.users[0].longitude = position.coords.longitude;
            // this.users[0].marker.setLngLat([position.coords.longitude, position.coords.latitude]);
            // this.map.setCenter({
            //     lng: position.coords.longitude, 
            //     lat: position.coords.latitude
            // })
            const clientData = {
                location: {    
                    latitude:  position.coords.latitude,
                    longitude: position.coords.longitude
                },
                action: 'MOVING'
            };
            this.connection.send(JSON.stringify(clientData));
        },
        updateMarkers() {
            //set marker
            for(const participant of this.participants) {
                if(participant.uuid in this.markers) {
                    // update location if already created
                    this.markers[participant.uuid].setLngLat([participant.location.longitude, participant.location.latitude]);
                }
                else {
                    // create marker
                    this.markers[participant.uuid] = new tt.Marker().setLngLat([participant.location.longitude, participant.location.latitude]).addTo(this.map);
                }
            }

            //clean up markers
            for(const uuid of Object.keys(this.markers)) {
                if(!this.participant.some(p => p.uuid === this.uuid)) {
                    this.markers[uuid].remove();
                    delete this.markers[uuid];
                }
            }
        }
    }
}
</script>