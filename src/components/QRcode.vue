<template>
  <div>
    <div v-if="result">last QR code: {{ result }}</div>
    <qrcode-stream :camera="camera" @decode="onDecode" @init="onInit">
      <div v-show="showScanConfirmation" class="scan-confirmation">
        <img src="../assets/checkmark.svg" alt="Checkmark" width="128px" />
      </div>
    </qrcode-stream>
    <div>{{ scanResults }}</div>
  </div>
</template>

<script>
import { QrcodeStream } from "vue-qrcode-reader";
import axios from "axios";

export default {
  name: "QRcode",
  components: { QrcodeStream },
  data() {
    return {
      camera: "auto",
      result: null,
      showScanConfirmation: false,
      scanResults: [],
    };
  },
  methods: {
    async onInit(promise) {
      try {
        await promise;
      } catch (e) {
        console.error(e);
      } finally {
        this.showScanConfirmation = this.camera === "off";
      }
    },

    async onDecode(content) {
      this.pause();
      this.result = content;
      if (!this.scanResults.some(res => res == content)) {
        this.postRequest(content);
      }
      this.scanResults.push(content);
      this.scanResults = this.scanResults.filter((item, i) => this.scanResults.indexOf(item) === i);
      await this.timeout(5);
      this.unpause();
    },

    unpause() {
      this.camera = "auto";
    },

    pause() {
      this.camera = "off";
    },

    timeout(ms) {
      return new Promise((resolve) => {
        window.setTimeout(resolve, ms);
      });
    },

    dateTime(){
      const today = new Date();
      return `${today.getFullYear()}/${today.getMonth() + 1}/${today.getDate()}`;
    },

    async postRequest(qrcode) {
      try {
        const data = await axios({
          method: 'post',
          url: 'https://script.google.com/macros/s/AKfycbwRP4d7RFgY2Z8yvUgwr9XGaopeCRxpq2qI5JGqW-X-SqQJpdJCMNmDzONfd-bEYjrDHQ/exec',
          headers: {
            'Content-Type' : 'text/plain'
          },
          data: `${this.dateTime()}, ${qrcode}`
        });
        return data;
      } catch(e) {
        console.error(e);
        return null;
      }
    },
  },
};
</script>

<style scoped>
.scan-confirmation {
  position: absolute;
  width: 100%;
  height: 100%;

  background-color: rgba(255, 255, 255, 0.8);

  display: flex;
  flex-flow: row nowrap;
  justify-content: center;
}
</style>
