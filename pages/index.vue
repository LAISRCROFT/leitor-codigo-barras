<template>
  <b-container class="bv-example-row" fluid>
    <b-row>
        <b-col cols="6" lg="6" class="col-center mt-5 box-center align_camera">
            <v-quagga :onDetected="onDetected" :onProcessed="onProcessed" :readerSize="readerSize" :aspectRatio="aspectRatio" :patchSize="patchSize" :readerTypes="['i2of5_reader']"></v-quagga>
        </b-col>
        <b-col cols="5" class="offset-1 mt-5">
            <b-row>
                <b-col cols="12">
                    <h3>CODE</h3>
                </b-col>
                <b-col cols="12">
                    <p> {{boleto.codigo_barras}} </p>
                </b-col>
                <b-col cols="12">
                    <p> Length: {{ length_boleto }} </p>
                </b-col>
                <b-col cols="12" v-if="boleto_invalido">
                    <p style="color: red;"> Boleto Inválido! </p>
                </b-col>
                <b-col cols="12" v-if="!boleto_invalido && length_boleto > 43">
                    <p style="color: green;"> Success! </p>
                </b-col>
                <b-col cols="12" v-if="codigo_barra_formatado">
                    <p><b>Código de Barras Formatado:</b> <br>{{codigo_barra_formatado}}  </p>
                </b-col>
                <b-col cols="12" v-if="linha_digitavel_boleto">
                    <p><b>Linha digitável Boleto:</b> <br>{{linha_digitavel_boleto}}  </p>
                </b-col>
                <b-col cols="12" v-if="hidden_button">
                    <b-button @click="openQuaggar">Start</b-button>
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
            width: 600,
            height: 200
        },
        aspectRatio: {
            min: 1,
            max: 2,
        },
        patchSize: "x-large",
        boleto: {
          codigo_barras: ''
        },
        hidden_button: false,
        boleto_invalido: false,
        length_boleto: '',
        codigo_barra_formatado: '',
        linha_digitavel_boleto: ''
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
      async onDetected(data){
        console.log(data)
        console.log(data.codeResult.code)
        this.boleto.codigo_barras = data.codeResult.code
        var linhadigitavel = await this.retornarLinhaDigitavel(data.codeResult.code)
      },
      async retornarLinhaDigitavel(codigoBarra) {
          let codigoBarraFormatado = codigoBarra.replace(/\./g, '').replace(/ /g, '')
          console.log('codigoBarraFormatado', codigoBarraFormatado)
          this.length_boleto = codigoBarraFormatado.length
          if (codigoBarraFormatado.length !== 44) {
              console.log("boleto inválido")
              this.boleto_invalido = true
              this.codigo_barra_formatado = ''
              this.linha_digitavel_boleto = ''
              return "boleto inválido"
          }
          else {
              this.boleto_invalido = false
              if (codigoBarraFormatado.slice(0, 1) !== '8') {
                  console.log("oi 1")
                  return this.linhaDigitavelBoleto(codigoBarraFormatado)
              }
              else {
                  console.log("")
                  console.log("oi 2")
                  console.log("")
                  return this.linhaDigitavelConsumo(codigoBarraFormatado)
              }
          }
      },
      async linhaDigitavelConsumo(codigoBarra) {
          console.log("")
          console.log("CODIGO_BARRA: ", codigoBarra)
          console.log("")
          let blocoUm = codigoBarra.slice(0, 11)
          let digitoVerificadorUm = await this.calcularVerificadorMod10(blocoUm)
          let blocoDois = codigoBarra.slice(11, 22)
          let digitoVerificadorDois = await this.calcularVerificadorMod10(blocoDois)
          let blocoTres = codigoBarra.slice(22, 33)
          let digitoVerificadorTres = await this.calcularVerificadorMod10(blocoTres)
          let blocoQuatro = codigoBarra.slice(33, 44)
          let digitoVerificadorQuatro = await this.calcularVerificadorMod10(blocoQuatro)

          let teste = blocoUm + " " + 
                      digitoVerificadorUm + " " + 
                      blocoDois + " " + 
                      digitoVerificadorDois + " " + 
                      blocoTres + " " + 
                      digitoVerificadorTres + " " + 
                      blocoQuatro + " " + 
                      digitoVerificadorQuatro

          console.log("")
          console.log('LinhaDigitavelConsumo: ', teste)
          console.log("")
          this.codigo_barra_formatado = teste
          this.hidden_button = true
          Quagga.stop()
          return teste
      },
      async linhaDigitavelBoleto(codigoBarra) {
          let codigoBanco = codigoBarra.slice(0, 3)
          let codigoMoeda = codigoBarra.slice(3, 4)
          let campoLivreBlocoUm = codigoBarra.slice(19, 24)
          let digitoVerificadorUm = await this.calcularVerificadorMod10(`${codigoBanco}${codigoMoeda}${campoLivreBlocoUm}`)
          let campoLivreBlocoDois = codigoBarra.slice(24, 34)
          let digitoVerificadorDois = await this.calcularVerificadorMod10(campoLivreBlocoDois)
          let campoLivreBlocoTres = codigoBarra.slice(34, 44)
          let digitoVerificadorTres = await this.calcularVerificadorMod10(campoLivreBlocoTres)
          let digitoVerificadorQuatro = codigoBarra.slice(4, 5)
          let fatorVencimento = codigoBarra.slice(5, 9)
          let valor = codigoBarra.slice(9, 19)

          

          let teste = codigoBanco + " "
              + codigoMoeda + " "
              + campoLivreBlocoUm + " "
              + digitoVerificadorUm + " "
              + campoLivreBlocoDois + " "
              + digitoVerificadorDois + " "
              + campoLivreBlocoTres + " "
              + digitoVerificadorTres + " "
              + digitoVerificadorQuatro + " "
              + fatorVencimento + " "
              + valor

          console.log('LinhaDigitavelBoleto')
          console.log(teste)
          this.linha_digitavel_boleto = teste
          this.hidden_button = true
          Quagga.stop()
          return teste
      },
      async calcularVerificadorMod10(codigoBarraSessao) {
          if (codigoBarraSessao === "") {
              return "";
          }
          let digitoVerificador = 0;
          let numeroPar = true;
          let i = codigoBarraSessao.length
          for (let i = codigoBarraSessao.length; i > 0; i--) {
              const digito = parseInt(codigoBarraSessao[i - 1]);
              if (numeroPar) {
                  digitoVerificador += Math.floor((digito * 2) / 10) + ((digito * 2) % 10);
              } else {
                  digitoVerificador += digito;
              }
              numeroPar = !numeroPar;
          }
          digitoVerificador = digitoVerificador % 10;
          if (digitoVerificador !== 0) {
              digitoVerificador = 10 - digitoVerificador;
          }
          return String(digitoVerificador);
      },
      openQuaggar(){
          this.hidden_button = false
          Quagga.init({
            inputStream : {
                name : "Live",
                type : "LiveStream",
                target: document.querySelector('#yourElement')    // Or '#yourElement' (optional)
            },
            decoder : {
                readers : ["i2of5_reader"]
            },
            locator: {
                patchSize: "x-large"
            }
        }, function(err) {
            if (err) {
                console.log(err);
                return
            }
            console.log("Initialization finished. Ready to start");
            Quagga.start();
        });
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