<template>
    <div id="map-container">
        <div id="map-div">{{ locationFetchMessage }}</div>
    </div>
</template>
<style>
    @import 'https://api.tomtom.com/maps-sdk-for-web/cdn/6.x/6.21.3/maps/maps.css';
    #map-container { height: 100vh; width: 100vw; }
    #map-div { width: 100%; height: 100%; }
</style>
<script>
// import tt from '@tomtom-international/web-sdk-maps'
import { default as ttServices } from '@tomtom-international/web-sdk-services';
import { default as tt } from '@tomtom-international/web-sdk-maps';

export default {
    props: ['connection', 'participants', 'userId', 'messages', 'showRouting'],
    data() {
        return {
            markers: {},
            map: null,
            mapAutoFit: true,
            mapAutoFitSuspensionTimeout: 10000,
            mapStartingZoom: 12,
            routing: {
                routes: [],
                enabled: false,
                weight: 3,
                backgroundWeight: 4,
                updateDelay: 5000
            },
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
        others() {
            return this.participants.filter(p => p.uuid !== this.userId);
        }
    },
    mounted() {
        navigator.geolocation.getCurrentPosition(this.initializeMap, err => {
            console.log(err.message);
            switch(err.code) {
                case err.PERMISSION_DENIED: this.locationFetchMessage = 'Please grant permission to see map.'; break;
                case err.POSITION_UNAVAILABLE: this.locationFetchMessage = 'Could not detect your location.'; break;
                case err.TIMEOUT: this.locationFetchMessage = 'Took a long time trying to get your spot. I gave up!'; break;
            }
            
            const dummyLocation = {
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
            }, 5000);
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
                zoom: this.mapStartingZoom
            });
            this.map.on('drag', () => {
                this.mapAutoFit = false;
                setTimeout(() => {
                    this.mapAutoFit = true;
                }, this.mapAutoFitSuspensionTimeout);
            });
            this.map.on('zoom', () => {
                this.mapAutoFit = false;
                setTimeout(() => {
                    this.mapAutoFit = true;
                }, this.mapAutoFitSuspensionTimeout);
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
            if(this.connection.readyState === WebSocket.OPEN) this.connection.send(JSON.stringify(clientData));
        },
        updateMarkers() {
            let usersWithLocation = 0;
            const bounds = new tt.LngLatBounds();

            //clean up markers of dropped off users
            for(const uuid of Object.keys(this.markers)) {
                const userExists = this.participants.some(p => p.uuid === uuid);
                if(!userExists) {
                    this.markers[uuid].remove();
                    delete this.markers[uuid];
                }
            }
            
            for(const participant of this.participants) {

                if(participant.location.latitude === 0 && participant.location.longitude === 0) continue;

                // set bounds
                if(participant.location.longitude !== 0 && participant.location.latitude !== 0) {
                    bounds.extend(tt.LngLat.convert([participant.location.longitude, participant.location.latitude]));
                    usersWithLocation++;
                }

                //set or update marker
                if(participant.uuid in this.markers) {
                    const marker = this.markers[participant.uuid]
                    // update location if already created
                    marker.setLngLat([participant.location.longitude, participant.location.latitude]);
                    if(marker.showPopup) {
                        let popupHtml = marker.popupMessage ? `<em>${participant.name} says:</em><br/><h2>${marker.popupMessage}</h2>` : `<em>${participant.name}</em><br/>`;
                        if(marker.hasSummary) {
                            const distanceAway = this.calculateDistanceAway(marker.distanceFromUser);
                            const timeAway = this.calculateTimeAway(marker.timeFromUser);
                            popupHtml += `<em>(${distanceAway}, ${timeAway} away)</em>`
                        }
                        marker.setPopup(new tt.Popup({ offset: {bottom: [0, -40]}, closeButton: false, closeOnClick: true, closeOnMove: false }).setHTML(popupHtml)).togglePopup();
                        marker.showPopup = false;
                    }
                }
                else {
                    // create marker
                    this.markers[participant.uuid] = new tt.Marker().setLngLat([participant.location.longitude, participant.location.latitude]).addTo(this.map);
                    // this.markers[participant.uuid].setPopup(new tt.Popup({ offset: {bottom: [0, -40]}, closeButton: false, closeOnClick: true, closeOnMove: false }).setHTML(`<em>${participant.name}</em>`)).togglePopup();
                    this.markers[participant.uuid].showPopup = true;
                    this.markers[participant.uuid].hasSummary = false;
                }
            }

            if(usersWithLocation > 1) {
                // show all markers on map
                if(this.mapAutoFit) this.map.fitBounds(bounds, { padding: 100} );
            }
            else {
                // nicely centers the user if they are the only ones left
                if(this.user.location.longitude !== 0 && this.user.location.latitude !== 0) {
                    this.map.easeTo({
                        center: tt.LngLat.convert([this.user.location.longitude, this.user.location.latitude]),
                        zoom: this.mapStartingZoom
                    });
                }
            }

            this.handleRouting();
        },
        handleRouting() {
            if(this.routing.enabled) {

                const batchItems = [];

                const user = this.user;

                for(const participant of this.others) {
                    batchItems.push({
                        locations: `${user.location.longitude},${user.location.latitude}:${participant.location.longitude},${participant.location.latitude}`,
                        uuid: participant.uuid
                    });
                }

                if(batchItems.length) {
                    ttServices.services.calculateRoute({
                        batchMode: 'sync',
                        key: process.env.VUE_APP_TOMTOM_API_KEY,
                        batchItems: batchItems
                    }).then(results => {
                        if(this.routing.enabled) {
                            results.batchItems.forEach((singleRoute, index) => {
                                if(singleRoute.error) return;
                                const routeGeoJson = singleRoute.toGeoJson();
                                const routeBackgroundLayerId = `route_background_${index}`;
                                const routeLayerId = `route_${index}`;

                                if(!this.map.getLayer(routeBackgroundLayerId) && !this.map.getLayer(routeLayerId)) {
                                    this.map.addLayer(this.buildStyle(routeBackgroundLayerId, routeGeoJson, 'black', this.routing.backgroundWeight)).addLayer(this.buildStyle(routeLayerId, routeGeoJson, 'red', this.routing.weight));
                                    this.routing.routes[index] = [routeBackgroundLayerId, routeLayerId];
                                }

                                const marker = this.markers[batchItems[index].uuid];

                                if(marker) {
                                    const summary = routeGeoJson.features[index].properties.summary;
                                    marker.hasSummary = true;
                                    marker.distanceFromUser = summary.lengthInMeters;
                                    marker.timeFromUser = summary.travelTimeInSeconds;
                                    marker.showPopup = true;
                                }
                            });
                        }
                    });
                }
            }
            else {
                //clean up routes
                this.routing.routes.forEach((child, index) => {
                    this.map.removeLayer(child[0]);
                    this.map.removeLayer(child[1]);
                    this.map.removeSource(child[0]);
                    this.map.removeSource(child[1]);
                    this.routing.routes.splice(index, 1);
                });

                for(const uuid of Object.keys(this.markers)) {
                    this.markers[uuid].hasSummary = false;
                }
            }
        },
        buildStyle(id, data, color, width) {
            return {
                'id': id,
                'type': 'line',
                'source': {
                    'type': 'geojson',
                    'data': data
                },
                'paint': {
                    'line-color': color,
                    'line-width': width
                },
                'layout': {
                    'line-cap': 'round',
                    'line-join': 'round'
                }
            }
        },
        calculateDistanceAway(value) {
            let unit = 'm';
            switch(true) {
                case value > 1000: value /= 1000; unit = 'km'; break;
            }
            value = Math.round(value);
            return `${value} ${unit}`;
        },
        calculateTimeAway(value) {
            let unit = 's';
            switch(true) {
                case value > 3600: value /= 3600; unit = 'hr'; break;
                case value > 60: value /= 60; unit = 'min'; break;
            }
            value = Math.round(value);
            return `${value} ${unit}`;
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
        },
        showRouting: {
            handler(value) {
                this.routing.enabled = value;
                this.handleRouting();
            }
        },
        'routing.routes.length': {
            handler(value) {
                console.log("ROUTING.ROUTES CHANGED", value);
                if(value > 0) console.log("ROUTING.ROUTES FILLED");
            }
        }
    }
}
</script>