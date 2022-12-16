<template>
    <v-card flat class="d-flex flex-column fill-height">
        <v-card-title>
            <v-row>
                <v-col>
                    Others: <b>{{ otherParticipants.length ? otherParticipants.join() : 'None' }}</b>
                </v-col>
                <v-col class="text-right">
                    You: <b>{{ name }}</b>
                </v-col>
            </v-row>
        </v-card-title>
        <v-card-title>
             <v-row>
                <v-col>
                    <v-btn color="error" @click="leaveRoom"><v-icon>mdi-arrow-left</v-icon>&nbsp;Leave</v-btn>
                </v-col>
                <v-col v-if="canToggleRouting" class="text-center">
                    <v-switch v-model="showRouting" label="Routing"></v-switch>
                </v-col>
                <v-col class="text-right">
                    <v-btn color="primary" @click="toggleMap">{{ showingMap ? 'Chat' : 'Map' }}&nbsp;<v-icon>{{ showingMap ? 'mdi-comment-multiple-outline' : 'mdi-map' }}</v-icon></v-btn>
                </v-col>
            </v-row>
        </v-card-title>
        <v-card-text class="flex-grow-1 overflow-y-auto">
            <friend-locator v-if="showingMap" :connection="connection" :participants="participants" :userId="userId" :messages="messages" :showRouting="showRouting" ></friend-locator>
            <div v-else id="container">
                <v-row v-for="(message, index) in allMessages" :key="index">
                    <v-col :class="{ 'd-flex flex-row-reverse': message.outgoing }">
                        <chat-bubble :message="message.text" :sender="message.sender" :outgoing="message.outgoing" ></chat-bubble>
                        <!-- <v-chip :dark="!message.outgoing" class="pa-4 mb-2 chat-bubble">
                            {{ message.text }}
                        </v-chip> -->
                    </v-col>
                </v-row>
            </div>
        </v-card-text>
        <v-card-text class="flex-shrink-1">
            <v-toolbar>
                <v-text-field rounded mx-2 v-model="currentMessage" label="Your message" type="text" no-details outlined hide-details @keyup.enter="sendMessage" />
                <v-btn fab dark mx-2 color="green" @click="sendMessage" ><v-icon dark>mdi-send</v-icon></v-btn>
            </v-toolbar>
        </v-card-text>
    </v-card>
</template>
<script>
import ChatBubble from './ChatBubble.vue'
import FriendLocator from './FriendLocator.vue'

export default {
    props: ['connection', 'userId', 'participants', 'messages', 'name'],
    data() {
        return {
            element: null,
            showingMap: false,
            currentMessage: '',
            newMessageReceived: false,
            showRouting: false
        }
    },
    components: {
        ChatBubble,
		FriendLocator
    },
    computed: {
        otherParticipants() {
           return this.participants.filter(p => p.uuid !== this.userId).map(p => p.name);
        },
        allMessages() {
            return this.messages.map(m => ({ outgoing: m.senderId === this.userId, text: m.text, sender: m.sender }));
        },
        canToggleRouting() {
            return this.participants.length > 1 && this.showingMap;
        }
    },
    updated() {
        if(this.newMessageReceived) {
            this.newMessageReceived = false;
            this.scrollToLastMessage();
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
        toggleMap() {
            this.showingMap = !this.showingMap;
        },
        scrollToLastMessage() {
            if(!this.showingMap) {
                const el = this.$el.querySelector('#container').lastElementChild;
                if(el) el.scrollIntoView({behavior: "smooth", block: "nearest", inline: "center", alignToTop: false });
            }
        }
    },
    watch: {
        messages: {
            immediate: true,
            handler: function() {
                this.newMessageReceived = true;
            }
        }
    }
}
</script>