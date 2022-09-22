<template>
  <div>
    <div v-if="result">last QR code: {{ result }}</div>
    <qrcode-stream :camera="camera" @decode="onDecode" @init="onInit">
      <div v-show="showScanConfirmation" class="scan-confirmation">
        <img src="../assets/checkmark.svg" alt="Checkmark" width="128px" />
      </div>
    </qrcode-stream>
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
      this.result = content;
      await this.postRequest(content);
      this.pause();
      await this.timeout(500);
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

    async postRequest(qrcode) {
      const data = await axios({
        method: "post",
        url: "https://script.google.com/macros/s/AKfycbwRP4d7RFgY2Z8yvUgwr9XGaopeCRxpq2qI5JGqW-X-SqQJpdJCMNmDzONfd-bEYjrDHQ/exec",
        headers: {},
        body: `2022/09/22, ${qrcode}`
      });
      console.log(data)
    }
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
