<template>
	<v-app>
		<v-container class="fill-height pa-0 ">
			<v-row v-if="connection" class="no-gutters elevation-4">
				<!-- <v-col cols="12" sm="3" class="flex-grow-1 flex-shrink-0" style="border-right: 1px solid #0000001f;">
					<v-responsive class="overflow-y-auto fill-height" height="500">
						<v-list subheader>
							<v-list-item-group v-model="activeChat">
								<template v-for="(item, index) in parents">
									<v-list-item :key="`parent${index}`" :value="item.id">
										<v-list-item-avatar color="grey lighten-1 white--text">
											<v-icon>
												chat_bubble
											</v-icon>
										</v-list-item-avatar>
										<v-list-item-content>
											<v-list-item-title v-text="item.title" />
											<v-list-item-subtitle v-text="'hi'" />
										</v-list-item-content>
										<v-list-item-icon>
											<v-icon :color="item.active ? 'deep-purple accent-4' : 'grey'">
												chat_bubble
											</v-icon>
										</v-list-item-icon>
									</v-list-item>
									<v-divider :key="`chatDivider${index}`" class="my-0" />
								</template>
							</v-list-item-group>
						</v-list>
					</v-responsive>
				</v-col> -->
				<v-col cols="auto" class="flex-grow-1 flex-shrink-0">
					<v-responsive class="overflow-y-hidden fill-height" width="100vw" height="100vh">
						<message-panel :name="name" :connection="connection" :messages="messages" :participants="participants" :userId="userId"></message-panel>
					</v-responsive>
				</v-col>
			</v-row>
			<!-- <join-room v-else></join-room> -->
			<v-card v-else>
				<v-card-title>
					Enter your name and room name. Then click Join.
				</v-card-title>
				<v-card-text class="flex-grow-1 overflow-y-auto" id="container">
					<v-text-field v-model="name" outlined placeholder="Your Name"></v-text-field>
					<v-text-field v-model="room" outlined placeholder="Room Name"></v-text-field>
				</v-card-text>
				<v-card-actions class="justify-center">
					<v-btn @click="join">Join {{ room }} as {{ userName }}</v-btn>
				</v-card-actions>
			</v-card>
		</v-container>
		<v-snackbar :timeout="snackbar.timeout" v-model="snackbar.display">{{ snackbar.notice }}</v-snackbar>
	</v-app>
</template>

<script>
	import MessagePanel from './components/MessagePanel.vue'
	// import JoinRoom from './components/JoinRoom.vue'
	export default {
		name: 'App',
		data() {
			return {
				connection: null,
				name: '',
				room: '',
				userId: 0,
				participants: [],
				messages: [],
				snackbar: {
					display: false,
					notice: '',
					timeout: 2000
				}
			}
		},
		components: {
			MessagePanel
			// JoinRoom
		},
		computed: {
			userName() {
				return this.name.trim() ? this.name : 'Anonymous';
			}
		},
		methods: {
			addMessage(message) {
				this.messages.push(message);
			},
			join() {
				const serverUrl = process.env.VUE_APP_CHAT_SERVER_URL;
				// const serverUrl = 'ws://ws-server-production.up.railway.app:443';
				this.connection = new WebSocket(`${serverUrl}?name=${this.name}&room=${this.room}`);
				this.connection.onerror = event => {
					console.log(`Failed to reach WebSocket server ${serverUrl}`, event);
				};
				this.connection.onmessage = event => {
					const data = JSON.parse(event.data);
					console.log(data.action);
					switch(data.action) {
						case 'UPDATE_PARTICIPANTS': this.participants = data.participants; break;
						case 'DISPLAY_CLIENT_MESSAGE': this.addMessage(data.message); break;
						case 'WELCOME': this.userId = data.id; break;
						case 'GOODBYE': this.connection = null; break;
						case 'FORBIDDEN': this.connection.close(); this.connection = null; break;
					}
					if(data.notice) {
						this.snackbar.notice = data.notice;
						this.snackbar.display = true;
					}
				}
			}
		}
	};
</script>