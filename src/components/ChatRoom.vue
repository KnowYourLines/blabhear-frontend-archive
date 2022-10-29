<template>
  <div class="column" v-if="leftRoom">
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
    You left the room. Refresh to rejoin.
  </div>
  <div v-else-if="userAllowed">
    <div class="column">
      <div>
        <img
          src="@/assets/icons8-home-48.png"
          @click="returnHome"
          @contextmenu="returnHomeNewTab"
          @contextmenu.prevent
          class="home-button"
        />
        <img
          v-if="!showMembers"
          src="@/assets/icons8-management-48.png"
          @click="showRoomMembers"
          @contextmenu.prevent
          class="show-members"
        />
        <img
          v-if="showMembers"
          src="@/assets/icons8-communication-50.png"
          @click="hideRoomMembers"
          @contextmenu.prevent
          class="show-chat"
        />
        <img
          v-if="shareable"
          src="@/assets/icons8-connect-48.png"
          @click="share"
          @contextmenu.prevent
          class="share-button"
        />
      </div>

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
      <div v-if="!showMembers" id="conversation">
        <div
          :class="{
            'mini-conversation-container': showRecordInterface,
            'conversation-container': !showRecordInterface,
          }"
          ref="messages"
          @scroll="onScroll"
        >
          <div
            v-for="message in messages"
            :key="message"
            class="bubble-container"
            :class="{ myMessage: message.creator__username === this.userId }"
          >
            <div class="bubble">
              <div class="message-timestamp">{{ message.created_at }}</div>
              <br />
              <div class="name">{{ message.creator__display_name }}:</div>
              <div class="message">{{ message.content }}</div>
            </div>
          </div>
        </div>
        <div v-if="showRecordInterface && messageToSend.length > 0">
          <div class="message-container">
            <textarea
              class="read-chat-input"
              v-model="messageToSend"
              readonly
            />
          </div>
        </div>
        <div v-if="!showRecordInterface" class="input-container">
          <div>
            <form @submit.prevent="askToRecord" id="chat-form">
              <textarea
                class="chat-input"
                v-model.trim="messageToSend"
                placeholder="Enter your message"
                required
              />
            </form>
          </div>
          <div>
            <button class="chat-send" type="submit" form="chat-form">
              Send
            </button>
          </div>
        </div>
        <div v-else>
          <div class="record-playback" v-if="!isRecording">
            <div v-if="recordingData.length == 0">
              <img
                src="@/assets/icons8-u-turn-to-left-50.png"
                @click="cancelSend"
                class="back-button"
              />
              <img
                src="@/assets/icons8-add-record-60.png"
                @click="addRecording"
                class="add-record"
              />
              <img
                src="@/assets/icons8-block-microphone-60.png"
                @click="sendMessage"
                class="no-record"
              />
            </div>
            <div class="playback" v-else>
              <img
                src="@/assets/icons8-checkmark-50.png"
                @click="approveRecorded"
                class="approve-recorded"
              />
              <img
                src="@/assets/icons8-microphone-60.png"
                @click="recordAudio"
                class="record-button"
              />
              <audio
                controls
                :src="recordedAudioUrl"
                controlsList="nodownload nofullscreen noremoteplayback"
              ></audio>
              <img
                src="@/assets/icons8-bin-48.png"
                @click="deleteRecorded"
                class="bin-button"
              />
            </div>
          </div>
          <div class="recording" v-else>
            <div>
              <img
                src="@/assets/icons8-pause-squared-48.png"
                @click="pauseRecording"
                class="pause-button"
              />
            </div>
          </div>
        </div>
      </div>
      <div v-else id="room-members">
        <br />
        <Toggle v-model="privateRoom" @change="updatePrivacy">
          <template v-slot:label="{ checked, classList }">
            <span :class="classList.label">{{
              checked ? "Private" : "Public"
            }}</span>
          </template>
        </Toggle>
        <br /><br />
        <div v-if="privateRoom">
          <div v-if="privateRoom && joinRequests.length > 0" id="requests">
            <span><br /><b>Users requesting to join:</b><br /><br /></span>
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
        <div id="members">
          <b>Room members:</b><br /><br />
          <span v-for="member in roomMembers" :key="member">
            {{ member }}<br />
          </span>
        </div>
      </div>
    </div>
  </div>
  <div class="column" v-else>
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
      leftRoom: false,
      displayName: null,
      editDisplayName: false,
      editableDisplayName: null,
      messages: [],
      messageToSend: "",
      page: 0,
      showMembers: false,
      showRecordInterface: false,
      audio: null,
      isRecording: false,
      recordingFile: null,
      recorder: null,
      recordingData: [],
      recordedAudioUrl: "",
    };
  },
  methods: {
    askToRecord: function () {
      this.showRecordInterface = true;
    },
    addRecording: function () {
      navigator.permissions.query({ name: "microphone" }).then((permission) => {
        if (permission.state === "granted") {
          if (!this.audio) {
            this.audio = navigator.mediaDevices.getUserMedia({ audio: true });
          }
          this.recordAudio();
        } else {
          this.audio = navigator.mediaDevices.getUserMedia({ audio: true });
        }
      });
    },
    showRoomMembers: function () {
      this.showMembers = true;
      if (this.isRecording) {
        this.pauseRecording();
      }
    },
    hideRoomMembers: function () {
      this.showMembers = false;
    },
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
    cancelSend: function () {
      this.showRecordInterface = false;
    },
    sendMessage: function (recordingUrl = null) {
      this.roomWebSocket.send(
        JSON.stringify({
          message: this.messageToSend,
          upload: recordingUrl,
          command: "send_message",
        })
      );
      this.messageToSend = "";
      this.showRecordInterface = false;
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
    recordAudio: function () {
      this.isRecording = true;
      this.isPlaying = false;
      if (!this.recorder || this.recorder.state == "inactive") {
        this.audio.then((stream) => {
          this.recorder = new MediaRecorder(stream);
          this.recorder.ondataavailable = (event) => {
            this.recordingData.push(event.data);
          };
          this.recorder.start(0); //0 for as little audio buffering as possible so recording starts immediately
        });
      } else {
        this.recorder.resume();
      }
    },
    pauseRecording: function () {
      if (this.recorder) {
        this.isRecording = false;
        this.recorder.pause();
        this.recordingFile = new Blob(this.recordingData, {
          type: "audio/ogg; codecs=opus",
        });
        this.recordedAudioUrl = window.URL.createObjectURL(this.recordingFile);
      }
    },
    deleteRecorded: function () {
      this.recorder.ondataavailable = () => {};
      this.recorder.stop();
      this.recordingFile = null;
      this.recordingData = [];
      this.recordingInSeconds = 0;
    },
    approveRecorded: function () {
      this.roomWebSocket.send(
        JSON.stringify({
          command: "fetch_upload_url",
        })
      );
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
      } else if (data.type == "left_room") {
        this.leftRoom = true;
      } else if ("display_name" in data) {
        this.displayName = data.display_name;
        this.editableDisplayName = data.display_name;
      } else if ("new_message" in data) {
        const newMessage = data.new_message;
        const messageContainer = this.$refs.messages;
        const wasAtBottom =
          messageContainer.getBoundingClientRect().bottom <=
            messageContainer.scrollTop + messageContainer.clientHeight &&
          messageContainer.scrollHeight ==
            Math.round(
              messageContainer.scrollTop + messageContainer.clientHeight
            );
        this.messages.push(newMessage);
        if (newMessage.creator__username == this.userId || wasAtBottom) {
          this.$nextTick(() => {
            const messageContainer = this.$refs.messages;
            messageContainer.scrollTop = messageContainer.scrollHeight;
          });
        }
      } else if ("messages" in data) {
        if (data.page > this.page) {
          this.messages.unshift(...data.messages);
          this.page = data.page;
          const oldScrollHeight = this.$refs.messages.scrollHeight;
          this.$nextTick(() => {
            this.$refs.messages.scrollTop =
              this.$refs.messages.scrollHeight - oldScrollHeight;
          });
        }
      } else if (data.type == "refresh_messages") {
        const maxPage = Math.max(Math.ceil(this.messages.length / 10), 1);
        this.messages = [];
        this.page = 0;
        this.roomWebSocket.send(
          JSON.stringify({
            page: maxPage,
            command: "fetch_messages_up_to_page",
          })
        );
      } else if (data.type == "refresh_display_name") {
        this.roomWebSocket.send(
          JSON.stringify({
            command: "fetch_display_name",
          })
        );
      } else if ("upload_url" in data) {
        const requestOptions = {
          method: "PUT",
          headers: { "Content-Type": "application/ogg" },
          body: this.recordingFile,
        };
        fetch(data.upload_url, requestOptions)
          .then((response) => {
            console.log(response);
            this.sendMessage(data.upload_url);
            this.deleteRecorded();
          })
          .catch((error) => console.log(error));
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
.message-container {
  display: flex;
  justify-content: center;
}
.approve-recorded {
  cursor: pointer;
  transition: 0.2s;
}
.approve-recorded:hover {
  transform: scale(1.1);
}
.back-button {
  padding: 6px 10px;
  cursor: pointer;
  transition: 0.2s;
}
.back-button:hover {
  transform: scale(1.1);
}
.add-record {
  padding: 6px 10px;
  cursor: pointer;
  transition: 0.2s;
}
.add-record:hover {
  transform: scale(1.1);
}
.no-record {
  padding: 6px 10px;
  cursor: pointer;
  transition: 0.2s;
}
.no-record:hover {
  transform: scale(1.1);
}
.record-button {
  cursor: pointer;
  transition: 0.2s;
}
.record-button:hover {
  transform: scale(1.1);
}
.pause-button {
  cursor: pointer;
  transition: 0.2s;
}
.pause-button:hover {
  transform: scale(1.1);
}
.bin-button {
  cursor: pointer;
  transition: 0.2s;
}
.bin-button:hover {
  transform: scale(1.1);
}
.play-button {
  cursor: pointer;
  transition: 0.2s;
}
.play-button:hover {
  transform: scale(1.1);
}
#members {
  word-break: break-all;
}
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
  border-radius: 70%;
  cursor: pointer;
}
.share-button:hover {
  background: #e0e0e0;
}
.input-container {
  display: flex;
  justify-content: center;
}
.record-playback {
  display: flex;
  justify-content: center;
}
.playback {
  transform: scale(0.9);
  display: flex;
  justify-content: center;
}
.recording {
  display: flex;
  justify-content: center;
}
.show-members {
  padding: 6px 10px;
  border-radius: 70%;
  cursor: pointer;
}
.show-members:hover {
  background: #e0e0e0;
}
.show-chat {
  padding: 6px 10px;
  border-radius: 70%;
  cursor: pointer;
}
.show-chat:hover {
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
.mini-conversation-container {
  margin: 0 auto;
  max-width: 400px;
  height: 520px;
  padding: 0 20px;
  border: 3px solid #f1f1f1;
  overflow: scroll;
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
  word-break: break-all;
}

.myMessage .bubble {
  background-color: #abf1ea;
  border: 2px solid #87e0d7;
  float: left;
}
.read-chat-input {
  padding: 12px 20px;
  margin: 8px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  margin-right: 5px;
  width: 300px;
  resize: none;
  height: 80px;
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
  resize: none;
  height: 80px;
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