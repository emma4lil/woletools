<template>
  <v-row>
    <v-col cols="4">
      <v-card>
        <v-card-title primary-title> Account: {{ username }} </v-card-title>
        <v-card-text> Status: {{ status }} </v-card-text>
        <v-divider></v-divider>
        <v-card-text>
          <v-text-field
            v-model="token"
            name="token"
            label="Token"
            single-line
            outline
          ></v-text-field>
          <v-btn @click="start()" block color="primary" dark>Connect</v-btn>
        </v-card-text>
      </v-card>
    </v-col>
    <v-col cols="4">
      <v-card>
        <v-card-title primary-title> Notifications - {{notifhist.length}} </v-card-title>
        <v-divider></v-divider>
        <v-card-text>
          <v-card class="ma-2" v-for="(notif, i) in notifhist" :key="i">
            <v-card-text>
              {{notif}}
            </v-card-text>
          </v-card>
        </v-card-text>
      </v-card>
    </v-col>
    <v-col cols="4">
      <v-card>
        <v-card-title primary-title> Chats Event Stream </v-card-title>
        <v-divider></v-divider>
        <v-card-text>
          <div class="d-flex flex-column">
            <chat-card :sender="username" :data="d" v-for="(d, i) in chathist" :key="i" />
          </div>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import ChatCard from "../components/ChatCard.vue";
export default {
  components: { ChatCard },
  data() {
    return {
      api: "https://wole-api.herokuapp.com/",
      username: "",
      notifconn: null,
      chatconn: null,
      token:"",
      chathist: [],
      notifhist: [],
      status: "null",
    };
  },
  mounted() {
    //notify
    this.notifconn = new signalR.HubConnectionBuilder()
      .withUrl(this.api + "hubs/push-notifications", {
        accessTokenFactory: () => this.token,
      })
      .configureLogging(signalR.LogLevel.Information)
      .build();
    this.notifconn.on("get-notification-msgs", (message) => this.notificationHandler(message));
    this.notifconn.on("get-notification-msg", (message) => this.notificationHandlerSingle(message));
    this.notifconn.on("user-login", (user) => this.userLoginHandler(user));
    //chat
    this.chatConn = new signalR.HubConnectionBuilder()
      .withUrl(this.api+ "hubs/chats", {
        accessTokenFactory: () => this.token,
      })
      .configureLogging(signalR.LogLevel.Information)
      .build();
    this.chatConn.on("get-offline-chats", (msg) =>
      this.offlineChatHandler(msg)
    );
    this.chatConn.on("new-chat-msg", (msg) => this.newChatHandler(msg));
    this.chatConn.on("user-data", (msg) => this.userProfileHandler(msg));
    this.notifconn.onclose(async () => {
      this.status = "disconnected!";
    });
  },
  methods: {
    async start() {
      try {
        this.status = "Attempting connection...";
        await this.notifconn.start();
        await this.chatConn.start();
        this.status = "Connected!";
      } catch (err) {
        this.status = "Connection failed:" + err;
        console.log(err);
      }
    },
    notificationHandler(payload) {
      this.notifhist = JSON.parse(payload);
      console.log(payload);
    },
     notificationHandlerSingle(payload) {
      this.notifhist.push(JSON.parse(payload));
      alert("New InApp Notification Recieved!")
    },
    userLoginHandler(payload) {
      alert(user + "logged in");
    },
    offlineChatHandler(payload) {
      this.chathist = JSON.parse(payload);
    },
    newChatHandler(payload) {
      this.chathist.push(JSON.parse(payload));
    },
    userProfileHandler(payload) {
      this.username = payload;
    },
  },
};
</script>