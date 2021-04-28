<template>
  <div class="container">
    <div class="row"></div>
    <div class="row">
      <div class="col-sm-12 col-md-6 col-lg-6">
        <h2>Capture Detection</h2>
        <!-- <code v-if="device">{{ device.label }}</code> -->
        <div class="border">
          <vue-web-cam
            ref="webcam"
            :device-id="deviceId"
            width="100%"
            @started="onStarted"
            @stopped="onStopped"
            @error="onError"
            @cameras="onCameras"
            @camera-change="onCameraChange"
          />
        </div>

        <div class="row">
          <div class="col-md-12">
            <!-- <select v-model="camera">
              <option>-- Select Device --</option>
              <option
                v-for="device in devices"
                :key="device.deviceId"
                :value="device.deviceId"
              >
                {{ device.label }}
              </option>
            </select> -->
          </div>
          <div class="col-md-12">
            <button
              type="button"
              class="btn btn-primary mt-3"
              @click="onCapture"
            >
              Capture Photo
            </button>
          <button type="button" class="btn btn-danger mt-3 ml-1" @click="clearDetect()">
              Clear
            </button> 
            <!-- <button type="button" class="btn btn-success" @click="onStart">
              Start Camera
            </button> --> 
          </div>
        </div>
      </div>
      <div class="col-sm-12 col-md-6 col-lg-6">
        <h2>Captured Image</h2>
        <figure class="figure">
          <img :src="img" class="img-responsive" />
        </figure>
      </div>
    
      <div class="col-lg-12 text-left mt-5">
        <h3>
          Result found
          {{
            detectResult &&
            detectResult.detected_objects &&
            detectResult.detected_objects.length > 0
              ? detectResult.detected_objects.length
              : 0
          }}
          objects
        </h3>
      </div>
      <div class="col-lg-12 text-left">
          <pulse-loader :loading="loading" :color="'#007bff'"></pulse-loader>
      </div>
      <div
        class="col-lg-4 col-md-6 col-sm-12 mt-4 mb-2"
        v-for="(item, index) in detectResult.detected_objects" :key="index"
        style="width: 27rem"
      >
        <div class="card mt-2">
          <div class="card-body">
            <h5 class="card-title">Name: {{ item.name }}</h5>
            <h6 class="card-subtitle mb-2 text-muted">
              Parent {{ item.parent }}
            </h6>
            <h5 class="card-title">Confidence: {{ item.confidence }}</h5>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { WebCam } from "vue-web-cam";
import PulseLoader from 'vue-spinner/src/PulseLoader.vue'
export default {
  name: "ImageDetect",
  components: {
    "vue-web-cam": WebCam,
    "pulse-loader":PulseLoader
  },
  data() {
    return {
      img: null,
      camera: null,
      deviceId: null,
      devices: [],
      detectResult: [],
      loading:false
    };
  },
  computed: {
    device: function () {
      return this.devices.find((n) => n.deviceId === this.deviceId);
    },
  },
  watch: {
    camera: function (id) {
      this.deviceId = id;
    },
    devices: function () {
      // Once we have a list select the first one
      const [first] = this.devices;
      if (first) {
        this.camera = first.deviceId;
        this.deviceId = first.deviceId;
      }
    },
  },
  methods: {
    clearDetect() {
      this.img = ""
      this.detectResult = []
    },
    sendImageDetect(base64) {
      this.loading = true;
      const nvision = require("@nipacloud/nvision");

      const objectDetectionService = nvision.objectDetection({
        apiKey:
          "cdb29f355cb4059995e054208f8cc73c327e9bbc3a0c290e7d88c58022f3e4f8a6c491cf8411c1b1291068c25c15042aac",
      });
      objectDetectionService
        .predict({
          rawData: base64.replace("data:image/jpeg;base64,", ""),
        })
        .then((result) => {
          // Outout the result object to console
          if(result && result.detected_objects.length > 0) {
            this.loading = false
            this.detectResult = result;
          }
        }).catch(()=> {
            this.loading = false
            this.detectResult = [];
        })
    },
    onCapture() {
      this.img = this.$refs.webcam.capture();
      this.sendImageDetect(this.img);
      // console.log(this.img)
    },
    onStarted(stream) {
      console.log("On Started Event", stream);
    },
    onStopped(stream) {
      console.log("On Stopped Event", stream);
    },
    onStop() {
      this.$refs.webcam.stop();
    },
    onStart() {
      this.$refs.webcam.start();
    },
    onError(error) {
      console.log("On Error Event", error);
    },
    onCameras(cameras) {
      this.devices = cameras;
      console.log("On Cameras Event", cameras);
    },
    onCameraChange(deviceId) {
      this.deviceId = deviceId;
      this.camera = deviceId;
      console.log("On Camera Change Event", deviceId);
    },
  },
};
</script>