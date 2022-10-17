<template>
  <div class="column-center" v-if="leftPublicRoom">
    <img
      src="@/assets/icons8-home-48.png"
      @click="returnHome"
      @contextmenu="returnHomeNewTab"
      @contextmenu.prevent
      class="home-button"
    /><img
      src="@/assets/icons8-update-left-rotation-50.png"
      @click="refreshPage"
      class="refresh-button"
    /><br /><br />
    You left a public room. Refresh to rejoin.
  </div>
  <div v-else-if="userAllowed">
    <div class="column-left">
      <div v-if="shareable">
        <img
          src="@/assets/icons8-share-30.png"
          @click="share"
          @contextmenu.prevent
          class="share-button"
        />
      </div>
      <img
        src="@/assets/icons8-home-48.png"
        @click="returnHome"
        @contextmenu="returnHomeNewTab"
        @contextmenu.prevent
        class="home-button"
      />
    </div>
    <div class="column-center">
      <br />
      <label for="name">Room Name:</label><br /><br />
      <div v-if="!editDisplayName">
        <strong>{{ displayName }}</strong
        ><br /><img
          src="@/assets/icons8-edit-24.png"
          @click="edit"
          class="edit-button"
        />
      </div>
      <div v-else>
        <input
          id="name"
          type="text"
          v-model="editableDisplayName"
          ref="editName"
        />
        <br /><img
          src="@/assets/icons8-checkmark-48.png"
          @click="updateDisplayName"
          class="edit-button"
        /><img
          src="@/assets/icons8-cancel-48.png"
          @click="cancelEdit"
          class="edit-button"
        />
      </div>
      <div id="conversation">
        <div class="conversation-container" ref="messages" @scroll="onScroll">
          <div
            v-for="message in messages"
            :key="message"
            class="bubble-container"
            :class="{ myMessage: message.creator__username === this.userId }"
          >
            <div class="bubble">
              <div class="message-timestamp">{{ message.created_at }}</div>
              <div class="name">{{ message.creator__display_name }}:</div>
              <div class="message">{{ message.content }}</div>
            </div>
          </div>
        </div>
        <div class="input-container">
          <input
            class="chat-input"
            @keyup.enter="sendMessage"
            v-model="messageToSend"
            placeholder="Enter your message"
          />
          <button class="chat-send" @click="sendMessage">Send</button>
        </div>
      </div>
      <br /><br /><br />
    </div>
    <div class="column-right">
      <br />
      <Toggle v-model="privateRoom" @change="updatePrivacy">
        <template v-slot:label="{ checked, classList }">
          <span :class="classList.label">{{
            checked ? "Private" : "Public"
          }}</span>
        </template>
      </Toggle>
      <br /><br />
      <div id="members">
        Room members:<br /><br />
        <span v-for="member in roomMembers" :key="member">
          {{ member }}<br />
        </span>
      </div>
      <div v-if="privateRoom">
        <span><br />Users requesting to join:<br /><br /></span>
        <div id="requests">
          <span v-for="request in joinRequests" :key="request.user">
            {{ request.user__display_name }}
            <div class="btn-group">
              <button
                type="button"
                class="btn"
                @click="acceptRequest(request.user__username)"
              >
                Accept
              </button>
              <button
                type="submit"
                class="btn btn__primary"
                @click="rejectRequest(request.user__username)"
              >
                Reject
              </button>
            </div>
            <br />
          </span>
        </div>
      </div>
    </div>
  </div>
  <div class="column-center" v-else>
    <img
      src="@/assets/icons8-home-48.png"
      @click="returnHome"
      @contextmenu="returnHomeNewTab"
      @contextmenu.prevent
      class="home-button"
    />
    <br />
    You are not allowed in private room. Access requested.
  </div>
</template>

<script>
import Toggle from "@vueform/toggle";
export default {
  name: "ChatRoom",
  components: {
    Toggle,
  },
  props: {
    room: {
      type: String,
      required: true,
    },
    authToken: {
      type: String,
      required: true,
    },
    userId: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      roomWebSocket: null,
      roomMembers: [],
      privateRoom: false,
      userAllowed: true,
      joinRequests: [],
      shareable: null,
      leftPublicRoom: false,
      displayName: null,
      editDisplayName: false,
      editableDisplayName: null,
      messages: [],
      messageToSend: "",
      page: null,
    };
  },
  methods: {
    returnHome: function () {
      const url = new URL(window.location.href);
      window.location.href = url.origin;
    },
    returnHomeNewTab: function () {
      const url = new URL(window.location.href);
      window.open(url.origin, "_blank");
    },
    refreshPage: function () {
      location.reload();
    },
    updatePrivacy: function () {
      if (!this.privateRoom) {
        this.roomWebSocket.send(
          JSON.stringify({ command: "approve_all_users" })
        );
      }
      this.roomWebSocket.send(
        JSON.stringify({
          command: "update_privacy",
          privacy: this.privateRoom,
        })
      );
    },
    acceptRequest: function (username) {
      this.roomWebSocket.send(
        JSON.stringify({ command: "approve_user", username: username })
      );
    },
    rejectRequest: function (username) {
      this.roomWebSocket.send(
        JSON.stringify({ command: "reject_user", username: username })
      );
    },
    share: function () {
      const shareData = {
        url: window.location.href,
      };
      navigator.share(shareData);
    },
    edit: function () {
      this.editDisplayName = true;
      this.$nextTick(() => {
        this.$refs.editName.select();
      });
    },
    cancelEdit: function () {
      this.editDisplayName = false;
      this.editableDisplayName = this.displayName;
    },
    updateDisplayName: function () {
      this.editDisplayName = false;
      this.roomWebSocket.send(
        JSON.stringify({
          command: "update_display_name",
          name: this.editableDisplayName,
        })
      );
    },
    sendMessage: function () {
      this.roomWebSocket.send(
        JSON.stringify({
          message: this.messageToSend,
          command: "send_message",
        })
      );
      this.messageToSend = "";
    },
    onScroll({ target: { scrollTop } }) {
      if (scrollTop == 0) {
        this.roomWebSocket.send(
          JSON.stringify({
            page: this.page + 1,
            command: "fetch_messages",
          })
        );
      }
    },
  },
  created() {
    this.shareable = typeof navigator.share === "function";
    const backendUrl = new URL(process.env.VUE_APP_BACKEND_URL);
    const ws_scheme = backendUrl.protocol == "https:" ? "wss" : "ws";
    const path =
      ws_scheme +
      "://" +
      backendUrl.hostname +
      ":" +
      backendUrl.port +
      "/ws/room/" +
      this.room +
      "/?token=" +
      this.authToken;
    this.roomWebSocket = new WebSocket(path);
    this.roomWebSocket.onopen = () => {
      console.log("Room WebSocket open");
    };
    this.roomWebSocket.onmessage = (message) => {
      const data = JSON.parse(message.data);
      if ("members" in data) {
        this.roomMembers = data.members;
      } else if (data.type == "refresh_members") {
        this.roomWebSocket.send(
          JSON.stringify({
            command: "fetch_members",
          })
        );
      } else if ("privacy" in data) {
        this.privateRoom = data.privacy;
      } else if (data.type == "refresh_privacy") {
        this.roomWebSocket.send(JSON.stringify({ command: "fetch_privacy" }));
      } else if ("allowed" in data) {
        this.userAllowed = data.allowed;
      } else if (data.type == "refresh_allowed_status") {
        this.roomWebSocket.send(
          JSON.stringify({ command: "fetch_allowed_status" })
        );
      } else if ("join_requests" in data) {
        this.joinRequests = data.join_requests;
      } else if (data.type == "refresh_join_requests") {
        this.roomWebSocket.send(
          JSON.stringify({
            command: "fetch_join_requests",
          })
        );
      } else if (data.type == "left_public_room") {
        this.leftPublicRoom = true;
      } else if ("display_name" in data) {
        this.displayName = data.display_name;
        this.editableDisplayName = data.display_name;
      } else if ("new_message" in data) {
        const newMessage = data.new_message;
        this.messages.push(newMessage);
        if (newMessage.creator__username == this.userId) {
          this.$nextTick(() => {
            const messageContainer = this.$refs.messages;
            messageContainer.scrollTop = messageContainer.scrollHeight;
          });
        }
      } else if ("messages" in data) {
        this.messages.unshift(...data.messages);
        this.page = data.page;
        if (this.page == 1) {
          this.$nextTick(() => {
            const messageContainer = this.$refs.messages;
            messageContainer.scrollTop = messageContainer.scrollHeight;
          });
        }
      } else if (data.type == "refresh_messages") {
        const maxPage = Math.ceil(this.messages.length / 10);
        this.messages = [];
        this.roomWebSocket.send(
          JSON.stringify({
            page: maxPage,
            command: "fetch_messages_up_to_page",
          })
        );
      }
    };
    this.roomWebSocket.onerror = (e) => {
      console.log(e.message);
    };
    this.roomWebSocket.onclose = () => {
      console.log("Room WebSocket closed");
      location.reload();
    };
  },
};
</script>

<style scoped>
#members {
  height: 15vh;
  overflow-y: auto;
  overflow-x: visible;
}
@media (orientation: landscape) {
  .column-left {
    float: left;
    width: 33.333%;
  }
  .column-right {
    float: right;
    width: 33.333%;
  }
  .column-center {
    display: inline-block;
    width: 33.333%;
  }
}
@media (orientation: portrait) {
  .column-left {
    width: 100%;
    background-color: rgb(227, 246, 255);
  }
  .column-right {
    width: 100%;
    background-color: rgb(227, 246, 255);
  }
  .column-center {
    width: 100%;
  }
}
.home-button {
  padding: 6px 10px;
  border-radius: 50%;
  cursor: pointer;
}
.home-button:hover {
  background: #e0e0e0;
}
.refresh-button {
  padding: 6px 10px;
  border-radius: 50%;
  cursor: pointer;
}
.refresh-button:hover {
  background: #e0e0e0;
}
.share-button {
  padding: 6px 10px;
  border-radius: 50%;
  cursor: pointer;
}
.share-button:hover {
  background: #e0e0e0;
}
.btn:hover {
  background: #e0e0e0;
}
.edit-button {
  padding: 6px 10px;
  border-radius: 50%;
  cursor: pointer;
}
.edit-button:hover {
  background: #e0e0e0;
}
.conversation-container {
  margin: 0 auto;
  max-width: 400px;
  height: 600px;
  padding: 0 20px;
  border: 3px solid #f1f1f1;
  overflow: scroll;
}

.bubble-container {
  text-align: left;
}

.message-timestamp {
  padding-right: 8px;
  font-size: 11px;
  float: right;
}

.bubble {
  border: 2px solid #f1f1f1;
  background-color: #fdfbfa;
  border-radius: 5px;
  padding: 10px;
  margin: 10px 0;
  width: 230px;
  float: right;
}

.myMessage .bubble {
  background-color: #abf1ea;
  border: 2px solid #87e0d7;
  float: left;
}
.chat-input {
  padding: 12px 20px;
  margin: 8px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  margin-right: 5px;
  width: 300px;
}
.chat-send {
  background-color: #21cfbc;
  color: white;
  padding: 14px 20px;
  margin: 8px 0;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.name {
  padding-right: 8px;
  font-size: 11px;
}

::-webkit-scrollbar {
  width: 10px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
  background: #888;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>