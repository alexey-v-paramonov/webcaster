<template>
  <div id="app">
    <b-button variant="success" v-on:click="startStream"
      >Start broadcasting</b-button
    >

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
  methods: {
    startStream: function () {
      console.log('Start stream')

      this.context = new AudioContext()
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
          'https://cdn.rawgit.com/webcast/libsamplerate.js/master/dist/libsamplerate.js',
          'https://cdn.rawgit.com/savonet/shine/master/js/dist/libshine.js',
          'https://cdn.rawgit.com/webcast/webcast.js/master/lib/webcast.js'
        ]
      })
      this.webcast = this.context.createWebcastSource(4096, 2)

      this.webcast.connectSocket(this.encoder, this.settings.uri);
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
