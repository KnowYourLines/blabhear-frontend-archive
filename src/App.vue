<template>
  <div v-if="!authToken">
    <SignIn @signed-in="signedIn" />
  </div>
  <div v-else-if="authToken && !room">
    <HomePage :authToken="authToken" :userId="userId" @new-room="newRoom" />
  </div>
  <div v-else>
    <ChatRoom :authToken="authToken" :room="room" :userId="userId" />
  </div>
</template>

<script>
import { v4 as uuidv4 } from "uuid";
import SignIn from "./components/SignIn.vue";
import HomePage from "./components/HomePage.vue";
import ChatRoom from "./components/ChatRoom.vue";

export default {
  name: "App",
  components: {
    SignIn,
    HomePage,
    ChatRoom,
  },
  data() {
    return {
      authToken: null,
      userId: null,
      room: null,
    };
  },
  methods: {
    signedIn: function (token, userId) {
      this.authToken = token;
      this.userId = userId;
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
  margin-top: 60px;
}
</style>
