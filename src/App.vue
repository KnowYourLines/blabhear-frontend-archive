<template>
  <div v-if="!authToken">
    <SignIn @signed-in="signedIn" />
  </div>
  <div v-else-if="authToken && !room">
    <div v-if="isAnonymous">
      <PhoneOnlySignIn @signed-in="signedIn" />
      <br />
    </div>
    <HomePage :authToken="authToken" :userId="userId" @new-room="newRoom" />
  </div>
  <div v-else>
    <div v-if="isAnonymous">
      <PhoneOnlySignIn @signed-in="signedIn" />
      <br />
    </div>
    <ChatRoom :authToken="authToken" :room="room" :userId="userId" />
  </div>
</template>

<script>
import SignIn from "./components/SignIn.vue";
import PhoneOnlySignIn from "./components/PhoneOnlySignIn.vue";
import HomePage from "./components/HomePage.vue";
import ChatRoom from "./components/ChatRoom.vue";

export default {
  name: "App",
  components: {
    SignIn,
    HomePage,
    ChatRoom,
    PhoneOnlySignIn,
  },
  data() {
    return {
      authToken: null,
      userId: null,
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
    newRoom: function (room) {
      const url = new URL(window.location.href);
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
