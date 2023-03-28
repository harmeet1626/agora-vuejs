<template>
  <div>
    <button id="join-btn" @click="() => joinStream()">Join Stream</button>

    <div id="stream-wrapper">
      <div id="video-streams"></div>

      <div id="stream-controls">
        <button id="leave-btn" @click="() => leaveAndRemoveLocalStream()">
          Leave Stream
        </button>
        <button id="mic-btn" @click="() => toggleMic()">Mic On</button>
        <button id="camera-btn" @click="() => toggleCamera()">Camera on</button>
      </div>
    </div>
  </div>
</template>
<script src="../../public/AgoraRTC_N-4.7.3.js"></script>
<script>
export default {
  data() {
    return {
      APP_ID: "6ff63c5227f04291a78decac94cae905",
      TOKEN:
        "007eJxTYCjdPPm9q/6kd44m8Xlrzuo1JWq9PuD5bNXtgmJWp6i150UVGMzS0syMk02NjMzTDEyMLA0TzS1SUpMTky1NkhNTLQ1My04ppTQEMjLcjzVmYWSAQBCfg8E5I7FEIbGggIEBAPr8IQs=",
      CHANNEL: "Chat app",
      client: "",
      localTracks: [],
      remoteUsers: {},
    };
  },
  async mounted() {
    this.client = await AgoraRTC.createClient({ mode: "rtc", codec: "vp8" });
  },
  methods: {
    async joinAndDisplayLocalStream() {
      // this.client.on("user-published", this.handleUserJoined);

      // this.client.on("user-left", this.handleUserLeft);

      let UID = await this.client.join(
        this.APP_ID,
        this.CHANNEL,
        this.TOKEN,
        null
      );

      this.localTracks = await AgoraRTC.createMicrophoneAndCameraTracks();

      let player = `<div class="video-container" id="user-container-${UID}">
                    <div class="video-player" id="user-${UID}"></div>
              </div>`;
      document
        .getElementById("video-streams")
        .insertAdjacentHTML("beforeend", player);

      this.localTracks[1].play(`user-${UID}`);

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
      delete remoteUsers[user.uid];
      document.getElementById(`user-container-${user.uid}`).remove();
    },

    async leaveAndRemoveLocalStream() {
      for (let i = 0; this.localTracks.length > i; i++) {
        this.localTracks[i].stop();
        this.localTracks[i].close();
      }

      await this.client.leave();
      document.getElementById("join-btn").style.display = "block";
      document.getElementById("stream-controls").style.display = "none";
      document.getElementById("video-streams").innerHTML = "";
    },
    async toggleMic(e) {
      if (this.localTracks[0].muted) {
        await localTracks[0].setMuted(false);
        // e.target.innerText = "Mic on";
        // e.target.style.backgroundColor = "cadetblue";
      } else {
        await this.localTracks[0].setMuted(true);
        // e.target.innerText = "Mic off";
        // e.target.style.backgroundColor = "#EE4B2B";
      }
    },
    async toggleCamera(e) {
      if (this.localTracks[1].muted) {
        await this.localTracks[1].setMuted(false);
        // e.target.innerText = "Camera on";
        // e.target.style.backgroundColor = "cadetblue";
      } else {
        await this.localTracks[1].setMuted(true);
        // e.target.innerText = "Camera off";
        // e.target.style.backgroundColor = "#EE4B2B";
      }
    },
  },
};
</script>
<style scoped>
body {
  background: #0f2027;
  background: -webkit-linear-gradient(to right, #2c5364, #203a43, #0f2027);
  background: linear-gradient(to right, #2c5364, #203a43, #0f2027);
}

#join-btn {
  position: fixed;
  top: 50%;
  left: 50%;
  margin-top: -50px;
  margin-left: -100px;
  font-size: 18px;
  padding: 20px 40px;
}

#video-streams {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
  height: 90vh;
  width: 1400px;
  margin: 0 auto;
}

.video-container {
  max-height: 100%;
  border: 2px solid black;
  background-color: #203a49;
}

.video-player {
  height: 100%;
  width: 100%;
}

button {
  border: none;
  background-color: cadetblue;
  color: #fff;
  padding: 10px 20px;
  font-size: 16px;
  margin: 2px;
  cursor: pointer;
}

#stream-controls {
  display: none;
  justify-content: center;
  margin-top: 0.5em;
}

@media screen and (max-width: 1400px) {
  #video-streams {
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    width: 95%;
  }
}
</style>
