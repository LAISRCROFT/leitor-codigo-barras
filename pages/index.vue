<template>
  <b-container class="bv-example-row" fluid>
    <b-row>
        <b-col cols="6" lg="6" class="col-center mt-5 box-center align_camera">
            <!-- <b-row> -->
                <!-- <b-col cols="10" class="align_camera"> -->
                    <v-quagga :onDetected="onDetected" :onProcessed="onProcessed" :readerSize="readerSize" :readerTypes="['i2of5_reader']"></v-quagga>
                <!-- </b-col> -->
            <!-- </b-row> -->
        </b-col>
        <b-col cols="5" class="offset-1 mt-5">
            <b-row>
                <b-col cols="12">
                    <h3>CODE</h3>
                </b-col>
            </b-row>
        </b-col>
    </b-row>
  </b-container>
</template>
<script>
  import Quagga from 'quagga';
  export default {
    name: 'leitor',
    data() {
      return {
        readerSize: {
            width: 640,
            height: 200
        },
      }
    },
    methods: {
      async onProcessed(result){
        var drawingCtx = Quagga.canvas.ctx.overlay,
              drawingCanvas = Quagga.canvas.dom.overlay;

          if (result) {
              if (result.boxes) {
                drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
                  result.boxes.filter(function (box) {
                      // console.log(result)
                      return box !== result.box;
                  }).forEach(function (box) {
                      Quagga.ImageDebug.drawPath(box, { x: 0, y: 1 }, drawingCtx, { color: "green", lineWidth: 2 });
                  });
              }

              if (result.box) {
                  Quagga.ImageDebug.drawPath(result.box, { x: 0, y: 1 }, drawingCtx, { color: "#00F", lineWidth: 2 });
              }

              if (result.codeResult && result.codeResult.code) {
                  Quagga.ImageDebug.drawPath(result.line, { x: 'x', y: 'y' }, drawingCtx, { color: 'red', lineWidth: 3 });
              }
          }
      },
      onDetected(data){
        console.log(data)
      }
    }
  }
</script>
<style scoped>
  .align_camera {
    
  }
 
  .viewport canvas, .viewport video {
    position: absolute;
    left: 0;
    top: 0;
    height: 265px;
}
</style>