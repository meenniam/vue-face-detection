<template>
  <div class="hello">
    <input type="file" accept="image/*" @change="fileChange" />
    <canvas ref="preview"></canvas>
    <!-- <canvas ref="preview2"></canvas> -->
    <!-- <canvas ref="preview3"></canvas> -->
    <color-picker v-model="colors"></color-picker>
  </div>
</template>

<script>
import * as faceapi from "face-api.js";
import { Compact } from "vue-color";
var colors = {
  hex: "#194d33",
  hsl: { h: 150, s: 0.5, l: 0.2, a: 1 },
  hsv: { h: 150, s: 0.66, v: 0.3, a: 1 },
  rgba: { r: 25, g: 77, b: 51, a: 1 },
  a: 1
};
export default {
  name: "HelloWorld",
  components: {
    "color-picker": Compact
  },
  props: {
    msg: String
  },
  data: () => ({
    context: null,
    canvas: null,
    image: null,
    context2: null,
    canvas2: null,
    context3: null,
    canvas3: null,
    detectionsWithLandmarks: null,
    fileBlob: null,
    fileName: "",
    colors,
    resizedResults: null
  }),
  mounted() {
    this.init();
    this.initCanvas();
    // this.initCanvas2();
    // this.initCanvas3();
  },
  watch: {
    colors() {
      this.changeColor();
    }
  },
  methods: {
    async init() {
      await faceapi.nets.ssdMobilenetv1.loadFromUri("/models");
      await faceapi.nets.faceLandmark68Net.loadFromUri("/models");
      await faceapi.loadFaceRecognitionModel("/models");
    },
    initCanvas() {
      this.$nextTick(() => {
        this.canvas = this.$refs.preview;
        this.context = this.canvas.getContext("2d");
        this.canvas.width = 400;
        this.canvas.height = 500;
        this.context.width = this.canvas.width;
        this.context.height = this.canvas.height;
      });
    },
    initCanvas2() {
      this.$nextTick(() => {
        this.canvas2 = this.$refs.preview2;
        this.context2 = this.canvas2.getContext("2d");
        this.canvas2.width = 400;
        this.canvas2.height = 500;
        this.context2.width = this.canvas2.width;
        this.context2.height = this.canvas2.height;
      });
    },
    initCanvas3() {
      this.$nextTick(() => {
        this.canvas3 = this.$refs.preview3;
        this.context3 = this.canvas3.getContext("2d");
        this.canvas3.width = 400;
        this.canvas3.height = 500;
        this.context3.width = this.canvas3.width;
        this.context3.height = this.canvas3.height;
      });
    },
    fileChange(e) {
      //// image canvas ////
      this.$nextTick(() => {
        this.fileName = e.target.files[0].name;
        const reader = new FileReader();
        reader.readAsDataURL(e.target.files[0]);
        reader.onload = this.readerLoad;
        reader.onerror = error => console.log(error);
      });
    },
    readerLoad(e) {
      //// image canvas ////
      this.image = new Image();
      this.image.src = e.target.result;
      this.image.onload = this.imageLoad;
    },
    imageLoad() {
      this.drawLandMark();
      // this.drawMouse();
      // this.changeImageColor();
      // context.canvas.toBlob(
      //   blob => {
      //     this.fileBlob = new File([blob], this.fileName, {
      //       type: "image/jpeg",
      //       lastModified: Date.now()
      //     });
      //   },
      //   "image/jpeg",
      //   1
      // );
    },
    changeColor() {
      const vm = this;
      const { context, resizedResults, colors } = vm;
      // const { context3, resizedResults, colors } = vm;
      const { rgba } = colors;
      const landmarks = resizedResults.landmarks;
      const positions = landmarks.positions;
      // mouse position is 49 to 68
      for (let i = 49; i < 68; i++) {
        console.log(positions[i].x, positions[i].y);
        let img = context.getImageData(positions[i].x, positions[i].y, 10, 10);
        // let img = context3.getImageData(positions[i].x, positions[i].y, 10, 10);
        let data = img.data;
        for (let d = 0; d < data.length; d += 4) {
          data[d + 0] = rgba.r;
          data[d + 1] = rgba.g;
          data[d + 2] = rgba.b;
          data[d + 3] = 255;
        }
        // data[3] = colors.rgba.a;
        context.putImageData(img, positions[i].x, positions[i].y);
        // context3.putImageData(img, positions[i].x, positions[i].y);
      }
    },
    changeImageColor() {
      const vm = this;
      const { context2, image, canvas2 } = vm;
      const { height, width } = context2;
      context2.drawImage(image, 0, 0, width, height);
      var imageData = context2.getImageData(
        0,
        0,
        canvas2.width,
        canvas2.height
      );
      this.changeToWhite(imageData.data);
      context2.putImageData(imageData, 0, 0);
    },
    async drawLandMark() {
      const vm = this;
      const { context, image } = vm;
      const { height, width } = context;
      const displaySize = { width, height };
      this.detectionsWithLandmarks = await faceapi
        .detectSingleFace(image)
        .withFaceLandmarks()
        .withFaceDescriptor();
      //// image canvas ////
      this.resizedResults = faceapi.resizeResults(
        this.detectionsWithLandmarks,
        displaySize
      );
      context.drawImage(image, 0, 0, width, height);
      faceapi.draw.drawFaceLandmarks(this.canvas, this.resizedResults);
      // this.contextLanmarkPutMouse(this.resizedResults);
    },
    async drawMouse() {
      const vm = this;
      const { context3, image } = vm;
      const { height, width } = context3;
      const displaySize = { width, height };
      context3.drawImage(image, 0, 0, width, height);
      this.detectionsWithLandmarks = await faceapi
        .detectSingleFace(image)
        .withFaceLandmarks()
        .withFaceDescriptor();
      //// image canvas ////
      this.resizedResults = faceapi.resizeResults(
        this.detectionsWithLandmarks,
        displaySize
      );
      context3.drawImage(image, 0, 0, width, height);
      // const landmarks = await faceapi.detectFaceLandmarks(canvas3);
      // const landmarkPositions = landmarks.positions;
      // console.log("mouse", landmarkPositions);
      // const mouse = landmarks.getMouth();
      // console.log("getMouse", mouse);
      // for (let i = 0; i < mouse.length; i++) {
      //   let img = context3.getImageData(mouse[i].x, mouse[i].y, 2, 2);
      //   let data = img.data;
      //   for (let d = 0; d < data.length; d++) {
      //     data[d] = 255;
      //   }
      //   context3.putImageData(img, mouse[i].x, mouse[i].y);
      // }
      // context.drawImage(image, 0, 0, width, height);
      // faceapi.draw.drawFaceLandmarks(this.canvas, resizedResults);
      // this.contextLanmarkPutImage(resizedResults);
    },
    contextLanmarkPutMouse(result) {
      const vm = this;
      const { context } = vm;
      const landmarks = result.landmarks;
      const positions = landmarks.positions;
      // mouse position is 49 to 68
      for (let i = 49; i < 68; i++) {
        let img = context.getImageData(positions[i].x, positions[i].y, 5, 5);
        let data = img.data;
        for (let d = 0; d < data.length; d++) {
          data[d] = 255;
        }
        context.putImageData(img, positions[i].x, positions[i].y);
      }
    },
    changeToWhite(data) {
      for (var i = 0; i < data.length; i += 4) {
        data[i] = 255 - data[i];
        data[i + 1] = 255 - data[i + 1];
        data[i + 2] = 255 - data[i + 2];
        data[i + 3] = 255;
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
