<template>
  <div v-if="!room">
    <div v-if="isAnonymous">
      <PhoneSignIn @signed-in="signedIn" />
    </div>
    <HomePage :authToken="authToken" :userId="userId" @new-room="newRoom" />
  </div>
  <div v-else>
    <ChatRoom :authToken="authToken" :room="room" :userId="userId" />
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
      isAnonymous: null,
    };
  },
  methods: {
    signedIn: function (token, userId, isAnonymous) {
      this.authToken = token;
      this.userId = userId;
      this.isAnonymous = isAnonymous;
    },
    newRoom: function () {
      const room = uuidv4();
      const url = new URL(window.location.href.split("?")[0]);
      url.searchParams.set("room", room);
      window.location.href = url;
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
