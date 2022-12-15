<template>
    <div>{{ users[0].latitude }},{{ users[0].longitude }}</div>
</template>
<script>
export default {
    data() {
        return {
            users: [
                {
                    latitude: 0,
                    longitude: 0
                }
            ]
        }
    },
    created() {
        if (navigator.geolocation) {
            navigator.geolocation.watchPosition(this.updatePosition);
        }
    },
    methods: {
        updatePosition(position) {
            this.users[0].latitude = position.coords.latitude;
            this.users[0].longitude = position.coords.longitude;
        }
    }
}
</script>