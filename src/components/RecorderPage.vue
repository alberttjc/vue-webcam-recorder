<template>
  <div class="recorder-page">

    <select class="camera-selection" v-model="deviceId">
      <option>--Select Device--</option>
        <option
            v-for="device in cameras"
            :key="device.deviceId"
            :value="device.deviceId"
            >{{device.label}}
        </option>
    </select>

    <app-media-recorder :videoStream="videoStream" :audioStream="audioStream" />

  </div>
</template>

<script>
import AppMediaRecorder from '@/components/AppMediaRecorder';

export default {
  name: 'RecorderPage',
  components: {
    AppMediaRecorder,
 },
  data() {
    return {
      audioStream: false,
      videoStream: false,

      deviceId: null,
      cameras: [],
    };
  },

  watch: {
    deviceId: function(id) {
      //this.cameraCheck();
      this.updateWebcam(id);
    }
  },

  methods: {
    testMediaAccess() {
      navigator.mediaDevices.getUserMedia({ audio: false,  video: { width: 640, height: 360 } }).then(stream=> {
        let tracks = stream.getTracks();
        tracks.forEach(track => {
          track.stop();
        });
        this.loadCameras();
      }).catch(error => this.$emit("error",error));
    },

    loadCameras() {
      navigator.mediaDevices.enumerateDevices().then(deviceInfos => {
        for (let i =0; i !== deviceInfos.length; ++i) {
          let deviceInfo = deviceInfos[i];
          if (deviceInfo.kind === "videoinput") {
            this.cameras.push(deviceInfo);
            //console.log(deviceInfo)
          }
        }
      })
    },

    getCameraStream(device) {
      let constraints = { audio: false,  video: { width: 640, height: 360, deviceId: device } };
      return navigator.mediaDevices.getUserMedia(constraints).catch(console.error);
    },

    updateWebcam: async function(device) {
      this.videoStream = await this.getCameraStream(device);
    },

    cameraCheck() {
      console.log(this.deviceId)
    },

  },

  async mounted() {
    this.testMediaAccess();
    this.videoStream = await this.getCameraStream(this.deviceId);
  },

}
</script>

<style scoped>
.recorder-page {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.camera-selection {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  font-size: 1.5em;

  padding: 5px 10px;
  border: 1px solid lightgray;
  border-radius: 10px;
  width: fit-content;
  margin-bottom: 20px;
}
</style>
