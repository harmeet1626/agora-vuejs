<template>
  <div class="about">
    <button @click="() => joinStream()" id="join-btn">Join Stream</button>

    <div id="stream-wrapper">
      <div id="video-streams"></div>
      <div id="stream-controls">
        <button @click="() => leaveAndRemoveLocalStream()" id="leave-btn">
          Leave Stream
        </button>
        <button @click="() => toggleMic()" id="mic-btn">Mic On</button>
        <button @click="() => toggleCamera()" id="camera-btn">Camera on</button>
      </div>
      <button @click="test">Test</button>
    </div>
  </div>
</template>
<script>
import AgoraRTC from "agora-rtc-sdk-ng";
export default {
  data() {
    return {
      APP_ID: "6ff63c5227f04291a78decac94cae905",
      TOKEN:
        "007eJxTYLDa9yfvyIFC+SO25bkNk0OWN7E3LU7MOlP5p8rm0Tqvld8VGMzS0syMk02NjMzTDEyMLA0TzS1SUpMTky1NkhNTLQ1M1YsVUxoCGRmmtn5hYWSAQBCfg8E5I7FEIbGggIEBAFQPIk4=",
      CHANNEL: "Chat app",
      client: AgoraRTC.createClient({ mode: "rtc", codec: "vp8" }),
      localTracks: [],
      remoteUsers: {},
      UID: "",
    };
  },
  async mounted() {
    this.UID = await this.client.join(
      this.APP_ID,
      this.CHANNEL,
      this.TOKEN,
      null
    );
  },
  methods: {
    test() {
      console.log(this.client)
    },
    async joinAndDisplayLocalStream() {
      // this.client.on("user-published", this.handleUserJoined);
      // this.client.on("user-left", this.handleUserLeft);

      this.localTracks = await AgoraRTC.createMicrophoneAndCameraTracks();
      let player = `<div class="video-container" id="user-container-${this.UID}">
                    <div class="video-player" id="user-${this.UID}"></div>
                    </div>`;
      document
        .getElementById("video-streams")
        .insertAdjacentHTML("beforeend", player);
      this.localTracks[1].play(`user-${this.UID}`);
      await this.client.publish([this.localTracks[0], this.localTracks[1]]);
    },
    async joinStream() {
      await this.joinAndDisplayLocalStream();
      document.getElementById("join-btn").style.display = "none";
      document.getElementById("stream-controls").style.display = "flex";
    },
    async handleUserJoined(user, mediaType) {
      this.remoteUsers[user.uid] = user;
      await this.client.subscribe(user, mediaType);

      if (mediaType === "video") {
        let player = document.getElementById(`user-container-${user.uid}`);
        if (player != null) {
          player.remove();
        }
        player = `<div class="video-container" id="user-container-${user.uid}">
                  <div class="video-player" id="user-${user.uid}"></div> 
                  </div>`;
        document
          .getElementById("video-streams")
          .insertAdjacentHTML("beforeend", player);

        user.videoTrack.play(`user-${user.uid}`);
      }
      if (mediaType === "audio") {
        user.audioTrack.play();
      }
    },
    async handleUserLeft(user) {
      delete this.remoteUsers[user.uid];
      document.getElementById(`user-container-${user.uid}`).remove();
    },
    async leaveAndRemoveLocalStream() {
      for (let i = 0; this.localTracks.length > i; i++) {
        this.localTracks[i].stop();
        this.localTracks[i].close();
      }
      await this.client.leave();
    },
    async toggleMic(e) {
      if (this.localTracks[0].muted) {
        await this.localTracks[0].setMuted(false);
        e.target.innerText = "Mic on";
        e.target.style.backgroundColor = "cadetblue";
      } else {
        await this.localTracks[0].setMuted(true);
        e.target.innerText = "Mic off";
        e.target.style.backgroundColor = "#EE4B2B";
      }
    },
    async toggleCamera(e) {
      if (this.localTracks[1].muted) {
        await this.localTracks[1].setMuted(false);
        e.target.innerText = "Camera on";
        e.target.style.backgroundColor = "cadetblue";
      } else {
        await this.localTracks[1].setMuted(true);
        e.target.innerText = "Camera off";
        e.target.style.backgroundColor = "#EE4B2B";
      }
    },
  },
};
</script>
