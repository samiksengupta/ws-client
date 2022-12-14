<template>
    <v-card flat class="d-flex flex-column fill-height">
        <v-card-title>
            {{ otherParticipants.join() }} Your Name: {{ name }}
        </v-card-title>
        <v-card-text class="flex-grow-1 overflow-y-auto" id="container">
            <v-btn @click="leaveRoom">Leave</v-btn>
            <hr>
            <div v-for="(message, index) in allMessages" :key="index" :class="{ 'd-flex flex-row-reverse': message.outgoing }">
                <!-- <v-chip :dark="!message.outgoing" class="pa-4 mb-2 chat-bubble">
                    {{ message.text }}
                </v-chip> -->
                <chat-bubble :message="message.text" :sender="message.sender" ></chat-bubble>
            </div>
        </v-card-text>
        <v-card-text class="flex-shrink-1">
            <v-text-field v-model="currentMessage" label="Your message" type="text" no-details outlined append-outer-icon="mdi-send" hide-details @keyup.enter="sendMessage" />
        </v-card-text>
    </v-card>
</template>
<style lang="scss" scoped>
    /* .chat-bubble {
        height: auto; 
        white-space: normal;
    } */
</style>
<script>
import ChatBubble from './ChatBubble.vue'

export default {
    props: ['connection', 'userId', 'participants', 'messages', 'name'],
    data() {
        return {
            currentMessage: ''
        }
    },
    components: {
        ChatBubble
    },
    updated() {
        this.scrollToLastMessage();
    },
    computed: {
        otherParticipants() {
           return this.participants.filter(p => p.id !== this.userId).map(p => p.name);
        },
        allMessages() {
            return this.messages.map(m => ({ outgoing: m.senderId === this.userId, text: m.text, sender: m.sender }));
        }
    },
    methods: {
        sendMessage() {
            const clientData = {
                message: {    
                    sender: this.name,
                    senderId: this.userId,
                    text: this.currentMessage
                },
                action: 'MESSAGING'
            };
            this.connection.send(JSON.stringify(clientData));
            this.currentMessage = '';
        },
        leaveRoom() {
            const clientData = {
                action: 'DISCONNECTING'
            };
            this.connection.send(JSON.stringify(clientData));
        },
        scrollToLastMessage() {
            const el = this.$el.querySelector('#container').lastElementChild;
            if(el) el.scrollIntoView({behavior: "smooth", block: "nearest", inline: "center", alignToTop: false });
        }
    }
}
</script>