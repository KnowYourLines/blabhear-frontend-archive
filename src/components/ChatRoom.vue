<template>
  <div class="column" v-if="leftRoom == room">
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
  <div v-else-if="userNotAllowedInRoom != room">
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
          v-if="!showRecordingSettings"
          src="@/assets/icons8-magic-wand-64.png"
          @click="showRoomRecordingSettings"
          @contextmenu.prevent
          class="show-magic"
        />
        <img
          v-if="showRecordingSettings"
          src="@/assets/icons8-communication-50.png"
          @click="hideRoomRecordingSettings"
          @contextmenu.prevent
          class="show-chat"
        />
      </div>

      <label for="name">Group Name:</label><br /><br />
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
      <div v-if="!showMembers && !showRecordingSettings" id="conversation">
        <div class="conversation-container" ref="messages" @scroll="onScroll">
          <div
            v-for="message in messages"
            :key="message"
            class="bubble-container"
            :class="{ myMessage: message.creator__username === this.userId }"
          >
            <div class="bubble">
              <div class="message-audio" v-if="message.filename != 'None'">
                <audio
                  :ref="message.filename + '-player'"
                  controls
                  :src="message.download"
                  controlsList="nodownload nofullscreen noremoteplayback"
                ></audio>
              </div>
              <div class="message-timestamp">{{ message.created_at }}</div>
              <div v-if="message.edited_at" class="edited-timestamp">
                Edited at {{ message.edited_at }}
              </div>
              <br />
              <div class="name">{{ message.creator__display_name }}:</div>
              <div
                class="message"
                v-if="
                  editMessageId != message.id &&
                  message.creator__username == userId
                "
              >
                <img
                  src="@/assets/icons8-edit-24.png"
                  @click="editMessage(message.id, message.content)"
                  class="edit-button"
                />{{ message.content }}
              </div>
              <div
                class="message"
                v-else-if="
                  editMessageId == message.id &&
                  message.creator__username == userId
                "
              >
                <form @submit.prevent="updateMessage(message.id)" id="edit-msg">
                  <textarea
                    class="edit-message"
                    v-model.trim="messageToEdit"
                    placeholder="Enter your message"
                    required
                  />
                </form>
                <button class="edit-msg-btn" type="submit" form="edit-msg">
                  <img
                    src="@/assets/icons8-checkmark-48.png"
                    class="edit-button"
                  />
                </button>
                <img
                  src="@/assets/icons8-cancel-48.png"
                  @click="cancelEditMessage"
                  class="edit-button"
                />
              </div>
              <div class="message" v-else>{{ message.content }}</div>
            </div>
          </div>
        </div>
        <div class="record-playback" v-if="!isRecording">
          <div v-if="recordingData.length == 0" class="input-container">
            <div>
              <form @submit.prevent="sendMessage" id="chat-form">
                <textarea
                  class="chat-input"
                  :value="messageToSend"
                  @input="
                    (event) => (messageToSend = event.target.value.trim())
                  "
                  placeholder="Enter your message"
                  required
                />
              </form>
            </div>
            <div v-if="messageToSend.length == 0">
              <img
                src="@/assets/icons8-microphone-60.png"
                @click="addRecording"
                class="add-record"
              />
            </div>
            <div v-else>
              <button class="chat-send" type="submit" form="chat-form">
                Send
              </button>
            </div>
          </div>
          <div
            class="playback"
            v-else-if="
              lastApprovedRecordedAudioUrl != wetRecordedAudioUrl &&
              wetRecordedAudioUrl
            "
          >
            <div>
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
                :src="wetRecordedAudioUrl"
                controlsList="nodownload nofullscreen noremoteplayback"
              ></audio>
              <img
                src="@/assets/icons8-bin-48.png"
                @click="deleteRecorded"
                class="bin-button"
              />
            </div>
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
      <div v-else-if="showMembers" id="room-members">
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
          <span><b>Users requesting to join:</b><br /><br /></span>
          <div v-if="joinRequests.length > 0" id="requests">
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
                <div class="divider" />
                <button
                  type="submit"
                  class="btn btn__primary"
                  @click="rejectRequest(request.user__username)"
                >
                  Reject
                </button>
              </div>
              <br
            /></span>
          </div>
          <div v-else>None</div>
          <br />
        </div>
        <img
          v-if="shareable"
          src="@/assets/icons8-add-users-48.png"
          @click="share"
          @contextmenu.prevent
          class="share-button"
        />
        <div id="members">
          <b>Group members:</b><br /><br />
          <span v-for="member in roomMembers" :key="member">
            {{ member }}<br />
          </span>
        </div>
      </div>
      <div v-else-if="showRecordingSettings">
        Transcribe to:
        <select v-model="chosenLanguage" @change="changeLanguage()">
          <option v-for="language in languages" :key="language">
            {{ language }}
          </option>
        </select>
        <br /><br />
        Voice effect:
        <select v-model="chosenVoiceEffect" @change="changeVoiceEffect()">
          <option v-for="effect in voiceEffects" :key="effect">
            {{ effect }}
          </option>
        </select>
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
import * as Tone from "tone";
import { bufferToWav } from "./bufferToWav";
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
    roomWebSocket: {
      type: WebSocket,
      required: false,
    },
    roomMembers: {
      type: Array,
      required: true,
    },
    privacy: {
      type: Boolean,
      required: true,
    },
    userNotAllowedInRoom: {
      type: String,
      required: true,
    },
    joinRequests: {
      type: Array,
      required: true,
    },
    leftRoom: {
      type: String,
      required: true,
    },
    displayName: {
      type: String,
      required: true,
    },
    messages: {
      type: Array,
      required: true,
    },
    page: {
      type: Number,
      required: true,
    },
    uploadDestination: {
      type: Object,
      required: true,
    },
    selectedLanguage: {
      type: String,
      required: true,
    },
    selectedVoiceEffect: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      shareable: null,
      privateRoom: false,
      editDisplayName: false,
      editableDisplayName: null,
      messageToSend: "",
      showMembers: false,
      audio: null,
      isRecording: false,
      recordingFile: null,
      wetRecordingFile: null,
      recorder: null,
      recordingData: [],
      recordedAudioUrl: "",
      wetRecordedAudioUrl: "",
      editMessageId: "",
      messageToEdit: "",
      lastApprovedRecordedAudioUrl: "",
      showRecordingSettings: false,
      chosenLanguage: "",
      chosenVoiceEffect: "",
      voiceEffects: [
        "None",
        "High Pitch",
        "Low Pitch",
        "Wobble",
        "Echo",
        "Fuzzy",
        "Speed Up",
        "Slow Down",
      ],
      languages: [
        "English",
        "English (GB)",
        "English (US)",
        "English (AU)",
        "English (IN)",
        "English (NZ)",
        "French",
        "French (CA)",
        "Ukrainian",
        "Spanish",
        "Spanish (LatAm)",
        "Turkish",
        "Tamil",
        "Swedish",
        "Russian",
        "Portuguese",
        "Portuguese (BR)",
        "Portuguese (PT)",
        "Polish",
        "Norwegian",
        "Korean",
        "Japanese",
        "Italian",
        "Indonesian",
        "Hindi",
        "Hindi (Roman)",
        "German",
        "Flemish",
        "Dutch/Flemish",
        "Danish",
        "Chinese",
        "Chinese (CN)",
        "Chinese (TW)",
      ],
    };
  },
  methods: {
    changeLanguage: function () {
      this.roomWebSocket.send(
        JSON.stringify({
          command: "change_language",
          language: this.chosenLanguage,
        })
      );
    },
    changeVoiceEffect: function () {
      this.roomWebSocket.send(
        JSON.stringify({
          command: "change_voice_effect",
          voice_effect: this.chosenVoiceEffect,
        })
      );
    },
    updateMessage: function (messageId) {
      this.roomWebSocket.send(
        JSON.stringify({
          command: "edit_message",
          edited_message: this.messageToEdit,
          message_id: messageId,
        })
      );
      this.editMessageId = "";
    },
    editMessage: function (messageId, message) {
      this.editMessageId = messageId;
      this.messageToEdit = message;
    },
    cancelEditMessage: function () {
      this.editMessageId = "";
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
      this.showRecordingSettings = false;
      if (this.isRecording) {
        this.pauseRecording();
      }
    },
    hideRoomMembers: function () {
      this.showMembers = false;
      this.showRecordingSettings = false;
    },
    showRoomRecordingSettings: function () {
      this.showRecordingSettings = true;
      this.showMembers = false;
      if (this.isRecording) {
        this.pauseRecording();
      }
    },
    hideRoomRecordingSettings: function () {
      this.showRecordingSettings = false;
      this.showMembers = false;
    },
    returnHome: function () {
      const url = new URL(window.location.href);
      window.history.replaceState("", "", url.origin);
      this.$emit("go-home");
      this.roomWebSocket.send(
        JSON.stringify({
          command: "disconnect",
        })
      );
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
      this.editableDisplayName = this.displayName;
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
    sendMessage: function (dryFilename, wetFilename) {
      this.roomWebSocket.send(
        JSON.stringify({
          message: this.messageToSend,
          dry_filename: dryFilename,
          wet_filename: wetFilename,
          command: "send_message",
        })
      );
      this.messageToSend = "";
    },
    onScroll({ target: { scrollTop } }) {
      if (scrollTop == 0 && this.page > 0) {
        this.roomWebSocket.send(
          JSON.stringify({
            page: this.page + 1,
            command: "fetch_messages",
          })
        );
      }
    },
    recordAudio: function () {
      Object.keys(this.$refs).filter((ref) => {
        if (ref.includes("player")) {
          if (!ref.paused && this.$refs[ref] && this.$refs[ref][0]) {
            this.$refs[ref][0].pause();
          }
        }
      });
      if (!this.recorder || this.recorder.state == "inactive") {
        this.audio.then((stream) => {
          this.recorder = new MediaRecorder(stream);
          this.recorder.onstart = () => {
            this.isRecording = true;
          };
          this.recorder.onresume = () => {
            this.isRecording = true;
          };
          this.recorder.onpause = () => {
            this.isRecording = false;
          };
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
        this.recorder.pause();
        this.recordingFile = new Blob(this.recordingData, {
          type: "audio/ogg; codecs=opus",
        });
        if (this.recordedAudioUrl) {
          window.URL.revokeObjectURL(this.recordedAudioUrl);
        }
        this.recordedAudioUrl = window.URL.createObjectURL(this.recordingFile);
        this.applyVoiceEffect();
      }
    },
    deleteRecorded: function () {
      this.recorder.ondataavailable = () => {};
      this.recorder.stop();
      this.recordingFile = null;
      this.recordingData = [];
    },
    approveRecorded: function () {
      this.lastApprovedRecordedAudioUrl = this.wetRecordedAudioUrl;
      this.roomWebSocket.send(
        JSON.stringify({
          command: "fetch_upload_url",
        })
      );
    },
    applyVoiceEffect: function () {
      let speedChange;
      if (this.chosenVoiceEffect == "Speed Up") {
        speedChange = 2;
      } else if (this.chosenVoiceEffect == "Slow Down") {
        speedChange = 0.5;
      }
      new Tone.Buffer(this.recordedAudioUrl, (toneAudioBuffer) => {
        Tone.Offline((context) => {
          const sample = new Tone.Player();
          sample.buffer = toneAudioBuffer;
          let voiceEffect;
          if (this.chosenVoiceEffect == "High Pitch") {
            voiceEffect = new Tone.PitchShift();
            voiceEffect.pitch = 5;
          } else if (this.chosenVoiceEffect == "Low Pitch") {
            voiceEffect = new Tone.PitchShift();
            voiceEffect.pitch = -5;
          } else if (this.chosenVoiceEffect == "Wobble") {
            voiceEffect = new Tone.Vibrato(10, 0.75);
          } else if (this.chosenVoiceEffect == "Echo") {
            voiceEffect = new Tone.FeedbackDelay("8n", 0.5);
          } else if (this.chosenVoiceEffect == "Fuzzy") {
            voiceEffect = new Tone.Distortion(0.75);
          } else {
            voiceEffect = new Tone.PitchShift();
            voiceEffect.pitch = 0;
            if (this.chosenVoiceEffect == "Speed Up") {
              sample.playbackRate = 2;
            } else if (this.chosenVoiceEffect == "Slow Down") {
              sample.playbackRate = 0.5;
            }
          }
          sample.chain(voiceEffect, context.destination);
          sample.start(0, 0);
        }, toneAudioBuffer.duration / speedChange).then((buffer) => {
          this.wetRecordingFile = bufferToWav(buffer, buffer.length);
          if (this.wetRecordedAudioUrl) {
            window.URL.revokeObjectURL(this.wetRecordedAudioUrl);
          }
          this.wetRecordedAudioUrl = window.URL.createObjectURL(
            this.wetRecordingFile
          );
        });
      });
    },
  },
  computed: {
    roomMessages() {
      return this.messages.slice();
    },
  },
  watch: {
    chosenVoiceEffect() {
      if (this.recordedAudioUrl) {
        this.applyVoiceEffect();
      }
    },
    selectedLanguage(newLanguage) {
      this.chosenLanguage = newLanguage;
    },
    selectedVoiceEffect(newEffect) {
      this.chosenVoiceEffect = newEffect;
    },
    privacy(newPrivacy) {
      this.privateRoom = newPrivacy;
    },
    roomMessages(newMessages, oldMessages) {
      if (
        newMessages.at(-1) != oldMessages.at(-1) &&
        newMessages.length == oldMessages.length + 1
      ) {
        const messageContainer = this.$refs.messages;
        const wasAtBottom =
          messageContainer.getBoundingClientRect().bottom <=
            messageContainer.scrollTop + messageContainer.clientHeight &&
          messageContainer.scrollHeight ==
            Math.round(
              messageContainer.scrollTop + messageContainer.clientHeight
            );
        const newMessage = this.messages.at(-1);
        if (newMessage.creator__username == this.userId || wasAtBottom) {
          this.$nextTick(() => {
            const messageContainer = this.$refs.messages;
            messageContainer.scrollTop = messageContainer.scrollHeight;
          });
        }
      } else if (newMessages.length > oldMessages.length) {
        const oldScrollHeight = this.$refs.messages.scrollHeight;
        this.$nextTick(() => {
          this.$refs.messages.scrollTop =
            this.$refs.messages.scrollHeight - oldScrollHeight;
        });
      }
    },
    uploadDestination(newUploadDestination) {
      const dryRequestOptions = {
        method: "PUT",
        headers: { "Content-Type": "audio/wav" },
        body: this.recordingFile,
      };
      fetch(newUploadDestination.dryUploadUrl, dryRequestOptions)
        .then(() => {
          const wetRequestOptions = {
            method: "PUT",
            headers: { "Content-Type": "audio/wav" },
            body: this.wetRecordingFile,
          };
          fetch(newUploadDestination.wetUploadUrl, wetRequestOptions)
            .then(() => {
              this.sendMessage(
                newUploadDestination.dryFilename,
                newUploadDestination.wetFilename
              );
              this.deleteRecorded();
            })
            .catch((error) => console.log(error));
        })
        .catch((error) => console.log(error));
    },
  },
  created() {
    this.shareable = typeof navigator.share === "function";
    this.privateRoom = this.privacy;
    this.chosenLanguage = this.selectedLanguage;
    this.chosenVoiceEffect = this.selectedVoiceEffect;
  },
};
</script>

<style scoped>
.input-container {
  display: flex;
  justify-content: center;
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
.divider {
  width: 50px;
  height: auto;
  display: inline-block;
}
.approve-recorded {
  cursor: pointer;
  transition: 0.2s;
}
.approve-recorded:hover {
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
  word-break: break-word;
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
.record-playback {
  display: flex;
  justify-content: center;
}
.playback {
  transform: scale(0.9);
  display: flex;
  justify-content: center;
  flex-direction: column;
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
.show-magic {
  border-radius: 70%;
  cursor: pointer;
}
.show-magic:hover {
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
.edited-timestamp {
  padding-right: 8px;
  font-size: 11px;
  float: right;
}

.message-audio audio::-webkit-media-controls-enclosure {
  max-width: 230px;
  margin: 10px 0;
  margin-right: 70px;
}

.bubble {
  border: 2px solid #f1f1f1;
  background-color: #fdfbfa;
  border-radius: 5px;
  padding: 10px;
  margin: 10px 0;
  width: 230px;
  float: right;
  word-break: break-word;
}

.myMessage .bubble {
  background-color: #abf1ea;
  border: 2px solid #87e0d7;
  float: left;
}
.edit-message {
  width: 230px;
  resize: none;
  height: 80px;
}
.edit-msg-btn {
  background-color: transparent;
  background-repeat: no-repeat;
  border: none;
  cursor: pointer;
  overflow: hidden;
  outline: none;
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