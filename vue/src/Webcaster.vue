<template>
  <div id="app">
    <b-button v-on:click="startStream" v-if="connectionState == 'disconnected'">
      Start broadcasting
    </b-button>

    <b-button variant="warning" v-else-if="connectionState == 'connecting'">
      Connecting...
    </b-button>

    <b-button variant="success" v-else-if="connectionState == 'connected'">
      Connected
    </b-button>

    <b-button variant="danger" v-else-if="connectionState == 'error'">
      Connection error
    </b-button>

    <b-form-input v-model="settings.uri" placeholder="Server URI"></b-form-input>

  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld'

export default {
  name: 'Webcaster',
  components: {
    // HelloWorld
  },
  data: () => {
    return {
      connectionState: 'disconnected',
      settings: {
        uri: 'ws://source:hackme@localhost:8080/mount',
        bitrate: 128,
        bitrates: [
          8,
          16,
          24,
          32,
          40,
          48,
          56,
          64,
          80,
          96,
          112,
          128,
          144,
          160,
          192,
          224,
          256,
          320
        ],
        samplerate: 44100,
        samplerates: [
          8000,
          11025,
          12000,
          16000,
          22050,
          24000,
          32000,
          44100,
          48000
        ],
        channels: 2,
        encoder: 'mp3',
        asynchronous: true,
        passThrough: false
      }
    }
  },
  created: function () {
    this.initAudioContext()
  },
  methods: {
    initAudioContext () {
      this.context = new AudioContext()
    },
    startStream: function () {
      console.log('Connecting to: ', this.settings.uri)

      this.context.resume()

      let Encoder = Webcast.Encoder.Mp3;
      this.encoder = new Encoder({
        channels: this.settings.channels,
        samplerate: this.settings.samplerate,
        bitrate: this.settings.bitrate
      })

      this.encoder = new Webcast.Encoder.Asynchronous({
        encoder: this.encoder,
        scripts: [
          'https://streaming.center/dist/webcaster/libsamplerate.js',
          'https://streaming.center/dist/webcaster/libshine.js',
          'https://streaming.center/dist/webcaster/webcast.js'
        ]
      })
      this.webcast = this.context.createWebcastSource(4096, 2)
      this.webcast.connectSocket(this.encoder, this.settings.uri)
      let that = this
      this.connectionState = 'connecting'
      let connectionAttemps = 0
      let connectionInterval = window.setInterval(function () {
        if (that.webcast.isOpen()) {
          that.connectionState = 'connected'
          clearInterval(connectionInterval)
          return
        }
        connectionAttemps += 1
        // No connection after 5 seconds
        if (connectionAttemps >= 50) {
          that.connectionState = 'error'
          clearInterval(connectionInterval)
        }
      }, 100)
    }
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
