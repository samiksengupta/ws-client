<template>
    <v-card>
        <v-card-title>
            Enter your name and room name. Then click Join.
        </v-card-title>
        <v-card-text class="flex-grow-1 overflow-y-auto" id="container">
            <v-text-field v-model="name" outlined placeholder="Your Name"></v-text-field>
            <v-text-field v-model="room" outlined placeholder="Room Name"></v-text-field>
        </v-card-text>
        <v-card-actions class="justify-center">
            <v-btn>Join {{ room }} as {{ userName }}</v-btn>
        </v-card-actions>
    </v-card>
</template>
<script>
export default {
    data() {
        return {
            name: '',
            room: ''
        }
    },
    computed: {
        userName() {
            return this.name.trim() ? this.name : 'Anonymous';
        }
    }
}
</script>