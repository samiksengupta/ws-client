<template>
    <div id="map-container">
        <div id="map-div">{{ locationFetchMessage }}</div>
    </div>
</template>
<style>
    @import 'https://api.tomtom.com/maps-sdk-for-web/cdn/6.x/6.21.3/maps/maps.css';
    #map-container { height: 55vh; width: 85vw; }
    #map-div { width: 100%; height: 100%; }
</style>
<script>
import tt from '@tomtom-international/web-sdk-maps'

export default {
    props: ['connection', 'participants', 'userId', 'messages'],
    data() {
        return {
            markers: {},
            map: null,
            locationFetchMessage: 'Trying to mark your location...'
        }
    },
    computed: {
        user() {
            let user = this.participants.find(p => p.uuid === this.userId);
            if(!user) {
                user = {
                    uuid: '',
                    name: '---',
                    location: {
                        latitude: 0,
                        longitude: 0
                    }
                }
            }
            return user;
        },
        
    },
    mounted() {
        navigator.geolocation.getCurrentPosition(this.initializeMap, err => {
            console.log(err.message);
            switch(err.code) {
                case err.PERMISSION_DENIED: this.locationFetchMessage = 'Please grant permission to see map.'; break;
                case err.POSITION_UNAVAILABLE: this.locationFetchMessage = 'Could not detect your location.'; break;
                case err.TIMEOUT: this.locationFetchMessage = 'Took a long time trying to get your spot. I gave up!'; break;
            }
            
            /* const dummyLocation = {
                coords: {
                    latitude: 22.634834,
                    longitude: 88.768484
                }
            }
            this.initializeMap(dummyLocation);
            setInterval(() => {
                dummyLocation.coords.latitude += 0.000001;
                dummyLocation.coords.longitude += 0.000001;
                this.updatePosition(dummyLocation);
            }, 2000); */
        });
        navigator.geolocation.watchPosition(this.updatePosition);
    },
    updated() {
    },
    methods: {
        requestPermission() {
            navigator.permissions.query({ name: 'geolocation' });
        },
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
        },
        updatePosition(position) {
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
                    if(this.markers[participant.uuid].showPopup) {
                        this.markers[participant.uuid].setPopup(new tt.Popup({ offset: {bottom: [0, -40]}, closeButton: false, closeOnClick: true, closeOnMove: false }).setHTML(`<em>${participant.name} says:</em><br/><h2>${this.markers[participant.uuid].popupMessage}</h2>`)).togglePopup();
                        this.markers[participant.uuid].showPopup = false;
                    }
                }
                else {
                    // create marker
                    this.markers[participant.uuid] = new tt.Marker().setLngLat([participant.location.longitude, participant.location.latitude]).addTo(this.map);
                    this.markers[participant.uuid].setPopup(new tt.Popup({ offset: {bottom: [0, -40]}, closeButton: false, closeOnClick: true, closeOnMove: false }).setHTML(`<em>${participant.name}</em>`)).togglePopup();
                    this.markers[participant.uuid].showPopup = false;
                }
            }

            //clean up markers
            for(const uuid of Object.keys(this.markers)) {
                const userExists = this.participants.some(p => p.uuid === uuid);
                if(!userExists) {
                    this.markers[uuid].remove();
                    delete this.markers[uuid];
                }
            }
        }
    },
    watch: {
        messages: {
            immediate: true,
            handler(newValue) {
                for(const message of newValue) {
                    if(message.text && this.markers[message.senderId]) {
                        this.markers[message.senderId].showPopup = true;
                        this.markers[message.senderId].popupMessage = message.text;
                    }
                }
            }
        },
        participants: {
            handler() {
                this.updateMarkers();
            }
        }
    }
}
</script>