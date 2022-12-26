<template>
  <div v-if="!room">
    <div v-if="isAnonymous">
      <div v-if="!verifyPhone">
        <img
          src="@/assets/icons8-decision-50.png"
          @click="verify"
          class="verify-button"
        />
      </div>
      <div v-else>
        <img
          src="@/assets/icons8-close-30.png"
          @click="cancelVerify"
          class="verify-button"
        />
        <PhoneSignIn @signed-in="signedIn" /><br />
      </div>
    </div>
    <div v-else>
      <img src="@/assets/icons8-checked-user-52.png" />
    </div>
    <HomePage
      :authToken="authToken"
      :userId="userId"
      :displayName="displayName"
      :notifications="notifications"
      :userWebSocket="userWebSocket"
      @new-room="newRoom"
      @visit-room="selectRoom"
    />
  </div>
  <div v-else>
    <ChatRoom
      :authToken="authToken"
      :room="room"
      :userId="userId"
      @go-home="goHome"
    />
  </div>
</template>

<script>
import { v4 as uuidv4 } from "uuid";
import firebase from "firebase/app";
import PhoneSignIn from "./components/PhoneSignIn.vue";
import HomePage from "./components/HomePage.vue";
import ChatRoom from "./components/ChatRoom.vue";

export default {
  name: "App",
  components: {
    HomePage,
    ChatRoom,
    PhoneSignIn,
  },
  data() {
    return {
      authToken: "",
      userId: "",
      room: null,
      isAnonymous: true,
      verifyPhone: false,
      notifications: [],
      displayName: "",
      userWebSocket: null,
    };
  },
  methods: {
    verify: function () {
      this.verifyPhone = true;
    },
    cancelVerify: function () {
      this.verifyPhone = false;
    },
    signedIn: function (token, userId, isAnonymous) {
      this.authToken = token;
      this.userId = userId;
      this.isAnonymous = isAnonymous;
    },
    newRoom: function () {
      const room = uuidv4();
      const url = new URL(window.location.href.split("?")[0]);
      url.searchParams.set("room", room);
      window.history.replaceState("", "", url);
      const urlParams = new URLSearchParams(window.location.search);
      this.room = urlParams.get("room");
    },
    selectRoom: function () {
      const urlParams = new URLSearchParams(window.location.search);
      this.room = urlParams.get("room");
    },
    goHome: function () {
      const urlParams = new URLSearchParams(window.location.search);
      this.room = urlParams.get("room");
    },
    connectWebsocket: function () {
      const backendUrl = new URL(process.env.VUE_APP_BACKEND_URL);
      const ws_scheme = backendUrl.protocol == "https:" ? "wss" : "ws";
      const path =
        ws_scheme +
        "://" +
        backendUrl.hostname +
        ":" +
        backendUrl.port +
        "/ws/user/" +
        this.userId +
        "/?token=" +
        this.authToken;
      this.userWebSocket = new WebSocket(path);
      this.userWebSocket.onopen = () => {
        console.log("User WebSocket open");
      };
      this.userWebSocket.onmessage = (message) => {
        const data = JSON.parse(message.data);
        if ("notifications" in data) {
          this.notifications = data.notifications;
        } else if (data.type == "refresh_notifications") {
          this.userWebSocket.send(
            JSON.stringify({
              command: "fetch_notifications",
            })
          );
        } else if ("display_name" in data) {
          this.displayName = data.display_name;
          this.editableDisplayName = data.display_name;
        }
      };
      this.userWebSocket.onerror = (e) => {
        console.log(e.message);
      };
      this.userWebSocket.onclose = () => {
        console.log("User WebSocket closed");
        this.connectWebsocket();
      };
    },
  },
  watch: {
    userId() {
      if (this.userWebSocket) {
        this.userWebSocket.close();
      } else {
        this.connectWebsocket();
      }
    },
  },
  mounted() {
    const urlParams = new URLSearchParams(window.location.search);
    this.room = urlParams.get("room");
    const firebaseConfig = {
      apiKey: process.env.VUE_APP_FIREBASE_API_KEY,
      authDomain: process.env.VUE_APP_FIREBASE_AUTH_DOMAIN,
      projectId: process.env.VUE_APP_FIREBASE_PROJECT_ID,
      storageBucket: process.env.VUE_APP_FIREBASE_STORAGE_BUCKET,
      messagingSenderId: process.env.VUE_APP_FIREBASE_MESSAGING_SENDER_ID,
      appId: process.env.VUE_APP_FIREBASE_APP_ID,
      measurementId: process.env.VUE_APP_FIREBASE_MEASUREMENT_ID,
    };
    firebase.initializeApp(firebaseConfig);
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        user.getIdToken().then((token) => {
          this.token = token;
          this.authToken = token;
          this.userId = user.uid;
          this.isAnonymous = user.isAnonymous;
        });
      } else {
        firebase.auth().signInAnonymously();
      }
    });
  },
};
</script>

<style src="@/assets/toggle.css"></style>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
<style scoped>
.verify-button {
  padding: 6px 10px;
  border-radius: 50%;
  cursor: pointer;
}
.verify-button:hover {
  background: #e0e0e0;
}
</style>