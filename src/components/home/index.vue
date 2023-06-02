<template>
  <div v-if="!ready"></div>
  <div v-else class="h-screen overflow-hidden w-full">
    <header class="flex justify-between bg-black p-4">
      <div class="nav--list">
        <button id="members__button">
          <svg
            width="24"
            height="24"
            xmlns="http://www.w3.org/2000/svg"
            fill-rule="evenodd"
            clip-rule="evenodd"
          >
            <path
              d="M24 18v1h-24v-1h24zm0-6v1h-24v-1h24zm0-6v1h-24v-1h24z"
              fill="#ede0e0"
            ></path>
            <path d="M24 19h-24v-1h24v1zm0-6h-24v-1h24v1zm0-6h-24v-1h24v1z" />
          </svg>
        </button>
        <a href="lobby.html">
          <h3 id="logo">
            <img src="@/assets/images/logo.png" alt="Site Logo" />
            <span>Mumble</span>
          </h3>
        </a>
      </div>

      <div id="nav__links">
        <button id="chat__button">
          <svg
            width="24"
            height="24"
            xmlns="http://www.w3.org/2000/svg"
            fill-rule="evenodd"
            fill="#ede0e0"
            clip-rule="evenodd"
          >
            <path
              d="M24 20h-3v4l-5.333-4h-7.667v-4h2v2h6.333l2.667 2v-2h3v-8.001h-2v-2h4v12.001zm-15.667-6l-5.333 4v-4h-3v-14.001l18 .001v14h-9.667zm-6.333-2h3v2l2.667-2h8.333v-10l-14-.001v10.001z"
            />
          </svg>
        </button>
        <!-- <a class="nav__link" href="/">
                Lobby
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="#ede0e0" viewBox="0 0 24 24"><path d="M20 7.093v-5.093h-3v2.093l3 3zm4 5.907l-12-12-12 12h3v10h7v-5h4v5h7v-10h3zm-5 8h-3v-5h-8v5h-3v-10.26l7-6.912 7 6.99v10.182z"/></svg>
            </a> -->
        <a class="nav__link" id="create__room__btn" href="lobby.html">
          Create Room
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            fill="#ede0e0"
            viewBox="0 0 24 24"
          >
            <path
              d="M12 0c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm6 13h-5v5h-2v-5h-5v-2h5v-5h2v5h5v2z"
            />
          </svg>
        </a>
      </div>
    </header>
    <div
      v-if="!join"
      class="r-wrap flex bg-white flex-col gap-3 justify-center items-center"
    >
      <p v-if="userData.uid && roomId">
        Welcome {{ displayName }} you are about to join {{ roomId }}
      </p>
      <p v-else>No room available</p>

      <div class="">
        <v-btn :disabled="!userData.uid && !roomId" @click="joinStream()"
          >Join</v-btn
        >
      </div>
    </div>
    <div v-else class="w-full grid grid-cols-5 r-wrap bg-slate-700">
      <div
        :id="`host-${hostuid}`"
        class="xs:w-full col-span-3 h-full main-screen bg-black border-2 w-full border-solid border-purple-500"
      ></div>
      <div
        class="grid place-content-start place-items-center col-span-2 p-5 grid-cols-2 w-full gap-5"
      >
        <div
          v-for="(i, index) in allUsers"
          :key="index"
          class="w-full h-auto border-2 rounded-md border-solid border-purple-500"
        >
          <div :id="`user-${i}`" class="w-full h-40"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import AgoraRTC from "agora-rtc-sdk-ng";
import AgoraRTM from "agora-rtm-sdk";

let client;
let rtmClient;
let channel;
// let members;
export default {
  name: "index",
  data: () => ({
    APP_ID: "cb16ef20c6c742208649235512a458c1",
    uid: "",
    customUsers: [
      { type: "host", uid: "emeter1" },
      { type: "user", uid: "emeter2" },
      { type: "user", uid: "emeter3" },
    ],
    roomId: "",
    token: null,
    userData: { uid: "", type: "" },
    displayName: "",
    localTracks: [],
    hostuid: "",
    localScreenTracks: [],
    sharingScreen: false,
    remoteUsers: {},
    allUsers: [],
    join: false,
    ready: false,
  }),
  methods: {
    async joinRoomInit() {
      rtmClient = await AgoraRTM.createInstance(this.APP_ID);
      await rtmClient.login({ uid: this.userData.uid, token: this.token });
      await rtmClient.addOrUpdateLocalUserAttributes({
        name: this.displayName,
      });
      channel = await rtmClient.createChannel(this.roomId);
      await channel.join();
      channel.on("MemberJoined", this.handleMemberJoined);
      // channel.on("MemberLeft", this.handleMemberLeft);
      // channel.on("ChannelMessage", this.handleChannelMessage);
      // this.getMembers();
      // this.addBotMessageToDom(`Welcome to the room ${this.displayName}! 👋`);
      client = AgoraRTC.createClient({ mode: "rtc", codec: "vp8" });
      await client.join(
        this.APP_ID,
        this.roomId,
        this.token,
        this.userData.uid
      );
      client.on("user-published", this.handleUserPublished);
      this.ready = true;
      // client.on("user-left", this.handleUserLeft);
    },
    async handleMemberJoined(MemberId) {
      console.log("A new member has joined the room:", MemberId);
      // this.addMemberToDom(MemberId);
      // members = await channel.getMembers();
      // this.updateMemberTotal(members);
      let { name } = await rtmClient.getUserAttributesByKeys(MemberId, [
        "name",
      ]);
      // addBotMessageToDom(`Welcome to the room ${name}! 👋`);
    },
    async addMemberToDom(MemberId) {
      let { name } = await rtmClient.getUserAttributesByKeys(MemberId, [
        "name",
      ]);
      let membersWrapper = document.getElementById("member__list");
      let memberItem = `<div class="member__wrapper" id="member__${MemberId}__wrapper">
                        <span class="green__icon"></span>
                        <p class="member_name">${name}</p>
                    </div>`;
      membersWrapper.insertAdjacentHTML("beforeend", memberItem);
    },

    async updateMemberTotal(members) {
      let total = document.getElementById("members__count");
      total.innerText = members.length;
    },

    async handleMemberLeft(MemberId) {
      this.removeMemberFromDom(MemberId);
      let members = await channel.getMembers();
      this.updateMemberTotal(members);
    },

    async removeMemberFromDom(MemberId) {
      let memberWrapper = document.getElementById(
        `member__${MemberId}__wrapper`
      );
      let name =
        memberWrapper.getElementsByClassName("member_name")[0].textContent;
      this.addBotMessageToDom(`${name} has left the room.`);

      memberWrapper.remove();
    },

    async getMembers() {
      let members = await channel.getMembers();
      this.updateMemberTotal(members);
      for (let i = 0; members.length > i; i++) {
        this.addMemberToDom(members[i]);
      }
    },

    async handleChannelMessage(messageData, MemberId) {
      console.log("A new message was received");
      let data = JSON.parse(messageData.text);

      if (data.type === "chat") {
        this.addMessageToDom(data.displayName, data.message);
      }

      if (data.type === "user_left") {
        document.getElementById(`user-container-${data.uid}`).remove();

        if (userIdInDisplayFrame === `user-container-${this.uid}`) {
          displayFrame.style.display = null;

          for (let i = 0; videoFrames.length > i; i++) {
            videoFrames[i].style.height = "300px";
            videoFrames[i].style.width = "300px";
          }
        }
      }
    },
    async sendMessage(e) {
      e.preventDefault();

      let message = e.target.message.value;
      channel.sendMessage({
        text: JSON.stringify({
          type: "chat",
          message: message,
          displayName: displayName,
        }),
      });
      this.addMessageToDom(displayName, message);
      e.target.reset();
    },

    async addMessageToDom(name, message) {
      let messagesWrapper = document.getElementById("messages");
      let newMessage = `<div class="message__wrapper">
                        <div class="message__body">
                            <strong class="message__author">${name}</strong>
                            <p class="message__text">${message}</p>
                        </div>
                    </div>`;

      messagesWrapper.insertAdjacentHTML("beforeend", newMessage);

      let lastMessage = document.querySelector(
        "#messages .message__wrapper:last-child"
      );
      if (lastMessage) {
        lastMessage.scrollIntoView();
      }
    },

    async addBotMessageToDom(botMessage) {
      let messagesWrapper = document.getElementById("messages");

      let newMessage = `<div class="message__wrapper">
                        <div class="message__body__bot">
                            <strong class="message__author__bot">🤖 Mumble Bot</strong>
                            <p class="message__text__bot">${botMessage}</p>
                        </div>
                    </div>`;

      messagesWrapper.insertAdjacentHTML("beforeend", newMessage);

      let lastMessage = document.querySelector(
        "#messages .message__wrapper:last-child"
      );
      if (lastMessage) {
        lastMessage.scrollIntoView();
      }
    },

    async leaveChannel() {
      await channel.leave();
      await rtmClient.logout();
    },
    async joinStream() {
      this.join = true;
      this.allUsers.push(this.userData.uid);
      this.localTracks = await AgoraRTC.createMicrophoneAndCameraTracks(
        {},
        {
          encoderConfig: {
            width: { min: 640, ideal: 1920, max: 1920 },
            height: { min: 480, ideal: 1080, max: 1080 },
          },
        }
      );
      this.localTracks[1].play(`user-${this.userData.uid}`);
      await client.publish([this.localTracks[0], this.localTracks[1]]);
    },

    async switchToCamera() {
      let player = `<div class="video__container" id="user-container-${this.uid}">
                    <div class="video-player" id="user-${this.uid}"></div>
                 </div>`;
      displayFrame.insertAdjacentHTML("beforeend", player);

      await localTracks[0].setMuted(true);
      await localTracks[1].setMuted(true);

      document.getElementById("mic-btn").classList.remove("active");
      document.getElementById("screen-btn").classList.remove("active");

      localTracks[1].play(`user-${this.uid}`);
      await client.publish([localTracks[1]]);
    },

    async handleUserPublished(user, mediaType) {
      this.remoteUsers[user.uid] = user;
      console.log(mediaType);
      console.log(user);
      let check = this.allUsers.includes(user.uid);
      if (!check) {
        this.allUsers.push(user.uid);
      }
      await client.subscribe(user, mediaType);
      if (mediaType === "video") {
        user.videoTrack.play(`user-${user.uid}`);
      }
      if (mediaType === "audio") {
        user.audioTrack.play();
      }
    },

    async handleUserLeft(user) {
      delete this.remoteUsers[user.uid];
      let item = document.getElementById(`user-container-${user.uid}`);
      if (item) {
        item.remove();
      }

      if (userIdInDisplayFrame === `user-container-${user.uid}`) {
        displayFrame.style.display = null;

        let videoFrames = document.getElementsByClassName("video__container");

        for (let i = 0; videoFrames.length > i; i++) {
          videoFrames[i].style.height = "300px";
          videoFrames[i].style.width = "300px";
        }
      }
    },

    async toggleMic(e) {
      let button = e.currentTarget;

      if (localTracks[0].muted) {
        await localTracks[0].setMuted(false);
        button.classList.add("active");
      } else {
        await localTracks[0].setMuted(true);
        button.classList.remove("active");
      }
    },

    async toggleCamera(e) {
      let button = e.currentTarget;

      if (localTracks[1].muted) {
        await localTracks[1].setMuted(false);
        button.classList.add("active");
      } else {
        await localTracks[1].setMuted(true);
        button.classList.remove("active");
      }
    },

    async toggleScreen(e) {
      let screenButton = e.currentTarget;
      let cameraButton = document.getElementById("camera-btn");

      if (!sharingScreen) {
        sharingScreen = true;

        screenButton.classList.add("active");
        cameraButton.classList.remove("active");
        cameraButton.style.display = "none";

        localScreenTracks = await AgoraRTC.createScreenVideoTrack();

        document.getElementById(`user-container-${this.uid}`).remove();
        displayFrame.style.display = "block";

        let player = `<div class="video__container" id="user-container-${this.uid}">
                <div class="video-player" id="user-${this.uid}"></div>
            </div>`;

        displayFrame.insertAdjacentHTML("beforeend", player);
        document
          .getElementById(`user-container-${this.uid}`)
          .addEventListener("click", expandVideoFrame);

        userIdInDisplayFrame = `user-container-${this.uid}`;
        localScreenTracks.play(`user-${this.uid}`);

        await client.unpublish([localTracks[1]]);
        await client.publish([localScreenTracks]);

        let videoFrames = document.getElementsByClassName("video__container");
        for (let i = 0; videoFrames.length > i; i++) {
          if (videoFrames[i].id != userIdInDisplayFrame) {
            videoFrames[i].style.height = "100px";
            videoFrames[i].style.width = "100px";
          }
        }
      } else {
        sharingScreen = false;
        cameraButton.style.display = "block";
        document.getElementById(`user-container-${this.uid}`).remove();
        await client.unpublish([localScreenTracks]);

        this.switchToCamera();
      }
    },

    async leaveStream(e) {
      e.preventDefault();

      document.getElementById("join-btn").style.display = "block";
      document.getElementsByClassName("stream__actions")[0].style.display =
        "none";

      for (let i = 0; localTracks.length > i; i++) {
        localTracks[i].stop();
        localTracks[i].close();
      }

      await client.unpublish([localTracks[0], localTracks[1]]);

      if (localScreenTracks) {
        await client.unpublish([localScreenTracks]);
      }

      document.getElementById(`user-container-${this.uid}`).remove();

      if (userIdInDisplayFrame === `user-container-${this.uid}`) {
        displayFrame.style.display = null;

        for (let i = 0; videoFrames.length > i; i++) {
          videoFrames[i].style.height = "300px";
          videoFrames[i].style.width = "300px";
        }
      }

      channel.sendMessage({
        text: JSON.stringify({ type: "user_left", uid: this.uid }),
      });
    },

    Join() {
      this.joinStream();
    },
  },
  created() {
    if (this.$route.query.room) {
      let currUser = this.customUsers.find(
        (v) => v.uid == this.$route.query.uid
      );
      this.userData = currUser;
      this.roomId = this.$route.query.room;
      this.displayName = this.$route.query.name;
      this.joinRoomInit();
    }
  },
};
</script>

<style>
@import "../../assets/css/lobby.css";
</style>
<style>
@import "../../assets/css/main.css";
</style>
<style>
@import "../../assets/css/room.css";
</style>
<style scoped>
header {
  height: 15vh;
}

.r-wrap {
  height: 85vh;
}
</style>
