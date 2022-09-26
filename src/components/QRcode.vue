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
      scanResults: [],
    };
  },
  async created() {
    const data = await this.getRequest();
    if (data){
      this.scanResults = data.map(item => item['QR Code']);
    }
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
      this.scanResults.push(content);
      this.scanResults = [...new Set(this.scanResults)]
      if (!this.scanResults.some(res => res === content)) {
        this.postRequest(content);
      }
      await this.timeout(10);
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

    async getRequest() {
      try {
        return (await axios({
          method: 'get',
          url: 'https://script.googleusercontent.com/macros/echo?user_content_key=V6Th2ArQPhnjoXe3QOg4QR_8h4fSzXKasVKVZVfhu_lxQyel0DJ6E7lNlTLMN7td0amoaywyoNFnW2-9knM9EbZWAGt7S9iAm5_BxDlH2jW0nuo2oDemN9CCS2h10ox_1xSncGQajx_ryfhECjZEnCsULUPfdPv4qksQcSQPjOjYePTlf0cJ843IY3onPE_sgkRfz4tDcIDZjPLnG_SNfje4f1p54VuRajchcNFUwCodxCCxmMaYptz9Jw9Md8uu&lib=MMFGFZz0UdlWm1ZkzaQuczxmnJgeV1HvH',
          headers: {
            'Content-Type' : 'text/plain'
          },
        })).data;
      } catch(e) {
        console.error(e);
        return null;
      }
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
