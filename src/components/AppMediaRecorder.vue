<template>
  <div class="media-recorder">
    <div class="video-cantainer" :style="videoContainerStyles">
      <video
        class="video"
        :controls="showVideo"
        ref="video"
        :width="videoSizes[0]"
        :height="videoHeight[1]"
        :src="recordUrl"
      />
    </div>

    <div class="controls">
      <div class="timer">
        <div v-if="this.webcam === 'nothing'">
          {{ timerStr[0] }}
        </div>
        <div v-if="this.webcam === 'start'">
          {{ timerStr[1] }} {{ timerCount}} {{ timerStr[3] }}
        </div>
        <div v-if="this.webcam === 'stop'">
          {{ timerStr[2] }} {{ timerCount}} {{ timerStr[3] }}
        </div>
      </div>

      <button v-if="showStartRecordingButton" class="button" @click="onStartPreview" title="Start recording">
        <img class="icon" src="../assets/icons/record.svg" />
      </button>
      <button v-else class="button" title="Stop">
        <img class="icon" src="../assets/icons/stop.svg" />
      </button>

      <button class="button" :disabled="downloadButtonDisabled" title="Download" @click="onDownloadRecord">
        <img class="icon" src="../assets/icons/download.svg" />
      </button>
    </div>

  </div>
</template>

<script>
const RECORDER_STATE = {
  RECORDING: 'recording',
  INACTIVE: 'inactive',
  PAUSED: 'paused',
};

const getCombinedStream = (...streams) => {
  const tracks = streams.map(stream => stream && stream.getTracks())
    .filter(val => !!val)
    .flat();
  return new MediaStream(tracks);
};

const STATE = {
  NOTHING: 'nothing',
  STARTRECORD: 'start',
  STOPRECORD: 'stop',
};

//import RecorderPage from '@/components/RecorderPage';

export default {
  name: 'AppMediaRecorder',
  props: {
    videoStream: {
      type: [MediaStream, Boolean],
      default: false,
    },
    audioStream: {
      type: [MediaStream, Boolean],
      default: false,
    },
    videoWidth: {
      type: Number,
      default: 1280,
    },
    videoHeight: {
      type: Number,
      default: 720,
    },
  },
  data() {
    return {
      recordBlob: null,
      recordVideo: true,
      recordAudio: false,
      recorderState: RECORDER_STATE.INACTIVE,

      // Timer
      timerStr: ["Please click the red button to start recording ","Recording starts in: ","Recording ends in: ", " seconds"],
      timerCount: 0,
      webcam: STATE.NOTHING,
    };
  },
  computed: {
    toggleVideoTitle() {
      return this.recordVideo ? 'Disable video recording' : 'Enable video recording';
    },
    toggleAudioTitle() {
      return this.recordAudio ? 'Disable audio recording' : 'Enable audio recording';
    },
    videoSizes() {
      const w = window.innerWidth;
      const h = window.innerHeight;
      const res = w / h;
      var width = Math.min(w, 800);
      var height = (width / res);

      if (width < 1280) {width = 1280}
      if (height < 720) {height = 720}

      return [ width, height ];
    },
    videoContainerStyles() {
      return {
        '--video-width': `${this.videoSizes[0]}px`,
        '--video-height': `${this.videoSizes[1]}px`,
      };
    },
    showVideo() {
      return this.recordUrl !== '';
    },
    combinedStream() {
      if (!this.waitForAudioStream && !this.waitForVideoStream) {
        return getCombinedStream(...[ this.recordAudio && this.audioStream, this.recordVideo && this.videoStream ]);
      }
      return null;
    },
    waitForAudioStream() {
      return this.recordAudio && !this.audioStream && this.audioStream !== false;
    },
    waitForVideoStream() {
      return this.recordVideo && !this.videoStream && this.videoStream !== false;
    },
    recorder() {
      if (!this.combinedStream) {
        return null;
      }
      const recorder = new MediaRecorder(this.combinedStream);
      recorder.ondataavailable = (dataObj) => this.recordBlob = dataObj.data;
      return recorder;
    },
    showStartRecordingButton() {
      return this.recorderState === RECORDER_STATE.INACTIVE;
    },
    showPauseRecordingButton() {
      return this.recorderState === RECORDER_STATE.RECORDING;
    },
    recordUrl() {
      if (this.recordBlob) {
        return URL.createObjectURL(this.recordBlob);
      }
      return '';
    },
    resumeButtonDisabled() {
      return this.recorderState !== RECORDER_STATE.PAUSED;
    },
    downloadButtonDisabled() {
      return !this.recordBlob;
    },
  },
  methods: {
    onToggleRecordAudio() {
      this.recordAudio = !this.recordAudio;
    },
    onToggleRecordVideo() {
      this.recordVideo = !this.recordVideo;
    },
    onStopRecording() {
      this.recorder.stop();
      this.recorderState = this.recorder.state;
      if (this.$refs.video.srcObject) {
        this.$refs.video.srcObject = null;
      }
    },
    onStartPreview() {
      this.webcam = STATE.STARTRECORD;
      this.recordBlob = null;
      this.timerCount = 5;

      this.$nextTick(() => {
          this.$refs.video.srcObject = this.videoStream;
          this.$refs.video.play();
      });
    },

    onStartRecording() {
      this.timerCount = 10;
      this.webcam = STATE.STOPRECORD

      this.$nextTick(() => {
        this.recorder.start();
        this.recorderState = this.recorder.state;

        if (this.recordVideo) {
          this.$refs.video.srcObject = this.videoStream;
          this.$refs.video.play();
        }

      });
    },

    onTimerRecording() {
      this.webcam = STATE.NOTHING
      this.recorder.stop();
      this.recorderState = this.recorder.state;
      if (this.$refs.video.srcObject) {
        this.$refs.video.srcObject = null;
      }
    },

    onPauseRecording() {
      this.recorder.pause();
      this.recorderState = this.recorder.state;
      if (this.recordVideo) {
        this.$refs.video.pause();
      }
    },
    onResumeRecording() {
      this.recorder.resume();
      this.recorderState = this.recorder.state;
      if (this.recordVideo) {
        this.$refs.video.play();
      }
    },
    onDownloadRecord() {
      if (this.recordUrl !== '') {
        const a = document.createElement('a');
        a.href = this.recordUrl;
        a.download = `0000-${Date.now()}.avi`;
        a.click();
      }
    },
    onTimerState() {
      if (this.webcam === STATE.STARTRECORD) {
        this.onStartRecording()
      }
      else if (this.webcam === STATE.STOPRECORD) {
        this.onTimerRecording()
      }
    },
    videoResolution(variable) {
      console.log(variable)
    },
  },

  watch: {
    timerCount: {
      handler(value) {

        if (value > 0) {
          setTimeout(() => {
            this.timerCount--;
          }, 1000);
        }
        else if (value < 0) {
          this.timerCount = 0;
        }

        if (this.timerCount === 0) {
          this.onTimerState()
        }
      },
      immediate: true,
    }
  },
}
</script>

<style scoped>
.video-cantainer {
  background-color: black;
  display: flex;
  justify-content: center;
  align-items: center;
  width: var(--video-width);
  height: var(--video-height);
  margin-bottom: 40px;
}
.controls {
  display: flex;
  justify-content: center;

  padding: 5px 10px;
  border: 1px solid lightgray;
}
.button {
  border: 1px solid lightgray;
  background-color: transparent;
  font-size: 1.5em;
  border-radius: 5px;
  cursor: pointer;
  display: flex;
  flex: 10%;
  justify-content: center;
  align-items: center;
  margin: 3px 4px;
  padding: 4px;
  min-width: 3em;
  width: fit-content;
  height: 1.8em;
}
.icon {
  display: block;
  height: 100%;
  max-width: 100%;
}
.button[disabled] {
  opacity: .5;
  cursor: not-allowed;
  outline: none;
}
.timer {
  display: flex;
  flex: 50%;
  justify-content: flex-start;
  padding: 5px 10px;

  font-size: 1.5em;
  font-weight: bold;
  margin: 5px 4px;
  padding: 4px;
  min-width: 3em;
  width: fit-content;
}
</style>
