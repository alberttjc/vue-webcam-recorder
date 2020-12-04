<template>
  <div class="recorder-page">

    <select class="camera-selection" v-model="deviceId">
      <option>--Select Device--</option>
        <option
            v-for="device in devices"
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

 props: {
   width: {
      type: [Number, String],
      default: "100%"
    },
    height: {
      type: [Number, String],
      default: 500
    },
    selectFirstDevice: {
      type: Boolean,
      default: false
    },
    resolution: {
      type: Object,
      default: null,
      validator: value => {
        return value.height && value.width;
      }
    }
 },
  data() {
    return {
      audioStream: false,
      videoStream: false,

      // Webcam selection
      deviceId: null,
      devices: [],
      cameras: [],
      camerasListEmitted: false,
    };
  },

  watch: {
    deviceId: function(id) {
      //this.cameraCheck();
      this.updateWebcam(id);
    },
    devices: function() {
        // eslint-disable-next-line no-unused-vars
        const [first, ...tail] = this.devices;
        if (first) {
            //this.camera = first.deviceId;
            this.deviceId = first.deviceId;
        }
    }
  },

  methods: {
    testMediaAccess() {

      let constraints = {video: true};

      if (this.resolution) {
        constraints.video = {};
        constraints.video.height = this.resolution.height;
        constraints.video.width = this.resolution.width;
      }
      navigator.mediaDevices.getUserMedia(constraints).then(stream=> {
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
            this.devices.push(deviceInfo);
            //console.log(deviceInfo)
          }
        }
      })
      // New code, delete if does not work
      .then(() => {
        if (!this.camerasListEmitted) {
          if (this.selectFirstDevice && this.devices.length > 0) {
            this.deviceId = this.devices[0].deviceId;
          }

          this.$emit("cameras", this.devices);
          this.camerasListEmitted = true;
        }
      })
      .catch(error => this.$emit("Not supported", error));

    },

    getCameraStream(device) {
      let constraints = { audio: false,  video: {deviceId: device } };

      constraints.video.height = 360;
      constraints.video.width = 640;
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
