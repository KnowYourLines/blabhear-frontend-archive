<template>
  <div>
    <div class="column">
      <label for="name">Name:</label><br /><br />
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
          maxlength="150"
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
      <br /><br />
      <button class="btn" @click="createNewRoom" @contextmenu.prevent>
        New group chat
      </button>
      <div>
        <span v-for="notification in notifications" :key="notification.room">
          <br />
          <div
            :class="notification.read ? 'notification' : 'unread-notification'"
            @click="visitRoom(notification.room)"
            @contextmenu="visitRoomNewTab(notification.room)"
            @contextmenu.prevent
          >
            <strong> {{ notification.room__display_name }}</strong>
            <br />
            <div v-if="notification.message__content">
              {{ notification.message__creator__display_name }}:
              {{ notification.message__content }}
            </div>
            <div v-else><br /></div>
            {{ notification.timestamp }} <br />
          </div>
          <button
            type="submit"
            class="btn btn__primary exit-btn"
            @click="exitRoom(notification.room)"
            @contextmenu.prevent
          >
            Exit group chat
          </button>
          <br />
        </span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HomePage",
  props: {
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
      userWebSocket: null,
      notifications: [],
      displayName: null,
      editDisplayName: false,
      editableDisplayName: null,
    };
  },
  methods: {
    createNewRoom: function () {
      this.$emit("new-room");
    },
    visitRoom: function (room) {
      const url = new URL(window.location.href);
      url.searchParams.set("room", room);
      window.location.href = url;
    },
    visitRoomNewTab: function (room) {
      const url = new URL(window.location.href);
      url.searchParams.set("room", room);
      window.open(url, "_blank");
    },
    exitRoom: function (room) {
      this.userWebSocket.send(
        JSON.stringify({ command: "exit_room", room_id: room })
      );
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
      this.userWebSocket.send(
        JSON.stringify({
          command: "update_display_name",
          name: this.editableDisplayName,
        })
      );
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
      if (this.userWebSocket) {
        this.userWebSocket.close();
      }
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
      };
    },
  },
  watch: {
    userId() {
      this.connectWebsocket();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@media (orientation: landscape) {
  .column {
    display: inline-block;
    width: 100%;
  }
}
@media (orientation: portrait) {
  .column {
    display: inline-block;
    width: 100%;
  }
}
.btn {
  background-color: #21cfbc;
  color: white;
  padding: 14px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.exit-btn {
  background-color: #10b981;
  color: white;
  padding: 14px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.notification {
  padding: 6px 10px;
  border-radius: 50%;
  border-style: solid;
  border-color: #10b981;
  cursor: pointer;
  overflow: clip;
}
.notification:hover {
  background: #e0e0e0;
}
.unread-notification {
  padding: 6px 10px;
  border-radius: 50%;
  border-style: solid;
  border-color: #10b981;
  background-color: rgb(76, 178, 247);
  cursor: pointer;
  overflow: clip;
}
.unread-notification:hover {
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
</style>