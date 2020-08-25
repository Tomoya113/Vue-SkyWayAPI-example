<template>
  <div>
    <video @loadeddata="onLoadedData" :srcObject.prop="myStream" id="my-video" width='400px' autoplay muted playsinline></video>
    <p id="my-id">
      {{ peerId }}
    </p>

    <textarea id="their-id" v-model="theirId"></textarea>
    <button id="make-call" @click="makeCall">発信</button>
    <video @loadeddata="onLoadedData" :srcObject.prop="theirStream" id="their-video" width="400px" autoplay muted playsinline></video>
  </div>
</template>

<script>
import Peer from 'skyway-js'
import { APIKEY } from '../key';
export default {
  name: 'Home',
  data() {
    return {
      peerId: "",
      theirId: "",
      localStream: null,
      peer: null,
      mediaConnection: null,
      myStream: null,
      theirStream: null,
    }
  },
  created() {
    navigator.mediaDevices.getUserMedia({ video: true, audio: true })
      .then(stream => {
        console.log({stream})
        this.myStream = stream
        this.localStream = stream;
      }).catch(error => {
        console.error('mediaDevice.getUserMedia() error:', error);
        return;
      })
    console.log(this.localStream)

    this.peer = new Peer({
      key: APIKEY,
      debug: 3,
    })

    this.peer.on('open', () => {
      console.log(this.peer.id)
      this.peerId = this.peer.id;
    })

    this.peer.on('call', mediaConnection => {
      mediaConnection.answer(this.localStream);
    })
  },
  methods: {
    makeCall() {
      console.log(this.theirId)
      this.mediaConnection = this.peer.call(this.theirId, this.localStream);
      this.mediaConnection.on('stream', stream => {
        this.theirStream = stream;
        console.log(this.theirStream)
        console.log(this.myStream);
      })
    },
    onLoadedData(event) {
      console.log(event)
      event.target.play();
    },
    // setEventListener(mediaConnection) {
      // mediaConnection.on('stream', stream => {
      //   this.theirStream = stream;
      //   console.log(this.theirStream)
      //   console.log(this.myStream);
      // })
    // }
  },
}
</script>
