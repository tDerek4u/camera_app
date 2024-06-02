<template>
  <div class="camera-container">
    <video ref="video" class="video" width="800" height="1200" autoplay></video>
    <button @click="takePhoto">Take a Shot</button>
    <canvas ref="canvas" class="hidden"></canvas>
    <img v-if="photo" :src="photo" class="photo" />
    <button v-if="photo" @click="printPhoto">Print</button>
    <a v-if="photo" @click="downloadPhoto">Download</a>
  </div>
</template>

<script>
export default {
  data() {
    return {
      photo: null,
    };
  },
  mounted() {
    this.startCamera();
  },
  methods: {
    startCamera() {
      navigator.mediaDevices
        .getUserMedia({ video: { aspectRatio: 2 / 3 } })
        .then((stream) => {
          this.$refs.video.srcObject = stream;
        })
        .catch((err) => {
          console.error("Error accessing the camera: ", err);
        });
    },
    takePhoto() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;
      // Set high resolution for the initial capture
      canvas.width = 1200;  
      canvas.height = 1800; 
      const ctx = canvas.getContext("2d");
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
      this.cropAndResizePhoto(canvas);
    },
    cropAndResizePhoto(canvas) {
      const finalOutput = document.createElement("canvas");
      const tempCtx = finalOutput.getContext("2d");
      finalOutput.width = 400;  
      finalOutput.height = 600; 

      const srcWidth = canvas.width;
      const srcHeight = srcWidth * (3 / 2);
      const srcX = 0;
      const srcY = (canvas.height - srcHeight) / 2;

      tempCtx.drawImage(
        canvas,
        srcX,
        srcY,
        srcWidth,
        srcHeight,
        0,
        0,
        finalOutput.width,
        finalOutput.height
      );

      this.photo = finalOutput.toDataURL("image/png");
    },
    printPhoto() {
      const printWindow = window.open("", "_blank");
      printWindow.document.write(
        `<img src="${this.photo}" onload="window.print();window.close()" />`
      );
      printWindow.document.close();
    },
    downloadPhoto() {
      const link = document.createElement("a");
      link.href = this.photo;
      link.download = "photo.png";
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
};
</script>

<style>
.camera-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.video {
  border: 1px solid black;
}
.hidden {
  display: none;
}
.photo {
  width: 400px;
  height: 600px;
  margin-top: 10px;
  border: 1px solid black;
}
button {
  margin-top: 10px;
}
</style>
