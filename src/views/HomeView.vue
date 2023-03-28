<template>
  <div class="app">
    <div class="video-container">
      <div id="local" class="video"></div>
      <div id="remote" class="video"></div>
    </div>
    <div class="chat-container">
      <div v-for="message in messages" :key="message.id">
        {{ message.content }}
      </div>
      <input v-model="inputMessage" @keydown.enter="sendMessage" />
      <button @click="() => sendMessage()">Send</button>
    </div>
  </div>
</template>

<script>
import AgoraRTC from "agora-rtc-sdk-ng";

export default {
  data() {
    return {
      client: null,
      channel: "Chat app",
      appId: "6ff63c5227f04291a78decac94cae905",
      token: '007eJxTYPjP+tFm+bO0abcK9i/aeVzY2btA99iPbvPnj7iEVX5O7udUYDBLSzMzTjY1MjJPMzAxsjRMNLdISU1OTLY0SU5MtTQwfe+smNIQyMhwwFqclZGBlYGRgYkBxGdgAAD8yx6F',
      localStream: null,
      remoteStream: null,
      messages: [],
      inputMessage: "",
    };
  },
  mounted() {
    this.initAgora();
  },
  methods: {
    async initAgora() {
      this.client = AgoraRTC.createClient({ mode: "rtc", codec: "vp8" });
      await this.client.join(this.appId, this.channel, this.token, null);
      this.localStream = AgoraRTC.createStream({
        streamID: this.client.uid,
        audio: true,
        video: true,
        screen: false,
      });
      await this.localStream.init();
      await this.client.publish(this.localStream);
      this.client.on("stream-added", async (evt) => {
        const remoteStream = evt.stream;
        this.remoteStream = remoteStream;
        await this.client.subscribe(remoteStream);
      });
      this.client.on("stream-subscribed", async (evt) => {
        const remoteStream = evt.stream;
        const remoteId = remoteStream.getId();
        remoteStream.play(`remote-${remoteId}`);
      });
      this.client.on("message", ({ account, text }) => {
        this.messages.push({ id: Date.now(), account, content: text });
      });
    },
    async sendMessage() {
      console.log(this.client);
      const { text } = await this.client.sendMessage(this.inputMessage);
      this.messages.push({
        id: Date.now(),
        account: "me",
        content: this.inputMessage,
      });
      this.inputMessage = "";
    },
  },
};
</script>
