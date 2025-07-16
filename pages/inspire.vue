<template>
  <v-container>
    <h1 class="mb-6 text-h4 font-weight-bold">QR Code Generator & Scanner</h1>

    <!-- QR Code Generator Section -->
    <v-card class="pa-4 mb-6">
      <v-text-field
        v-model="qrText"
        label="Enter text to generate QR"
        outlined
        dense
        clearable
      />
      <v-btn color="primary" @click="generateQRCode">Generate QR Code</v-btn>

      <div v-if="qrDataUrl" class="mt-4 text-center">
        <p class="mb-2">Generated QR Code:</p>
        <img :src="qrDataUrl" alt="QR Code" style="width: 250px; height: 250px;" />
      </div>
    </v-card>

    <!-- QR Code Scanner Section -->
    <v-card class="pa-4">
      <p class="mb-4 font-weight-medium">QR Code Scanner</p>
      <div id="reader" style="width: 100%; max-width: 500px; height: 500px; margin: 0 auto;"></div>

      <div v-if="scannedResult" class="mt-4 text-center">
        <p><strong>Scanned Result:</strong> {{ scannedResult }}</p>
      </div>
    </v-card>
  </v-container>
</template>

<script>
import QRCode from 'qrcode'

export default {
  data() {
    return {
      qrText: '',
      qrDataUrl: '',
      scannedResult: ''
    }
  },
  mounted() {
    if (!this.$html5qrcode) {
      console.error("html5-qrcode plugin not available.");
      return;
    }

    this.$html5qrcode.getCameras()
      .then(devices => {
        if (devices.length === 0) {
          console.error("No camera found.");
          return;
        }

        const cameraId = devices[0].id;
        const html5QrCode = new this.$html5qrcode("reader");

        html5QrCode.start(
          { deviceId: { exact: cameraId } },
          { fps: 10, qrbox: { width: 300, height: 300 } },
          (decodedText) => {
            this.scannedResult = decodedText
          },
          (errorMessage) => {
            // Handle errors if needed
          }
        ).catch(err => {
          console.error("Start failed:", err);
        });
      })
      .catch(err => {
        console.error("Error getting cameras:", err);
      });
  },
  methods: {
    async generateQRCode() {
      if (!this.qrText.trim()) {
        alert("Please enter some text");
        return;
      }

      try {
        this.qrDataUrl = await QRCode.toDataURL(this.qrText)
      } catch (error) {
        console.error("QR code generation failed:", error)
      }
    }
  }
}
</script>
