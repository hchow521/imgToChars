<template>
  <div id="app">
    <input id="selectImg" type="file" accept="image/*" @input="getImg" style="width:0">
    <button><label for="selectImg">请选择图片</label></button>
    <br/>
    <span title="越小越白哦">黑白曝光值：</span><input v-model="threshold" type="range" min="0" max="255" @change="imgOnload">{{threshold}}
    <br/>
    <span title="越大越清晰哦">生成图片大小：</span><input v-model="ascii_width" type="number" step="10" @change="imgOnload">
    <br/>
    <button @click="innerHTML=''">清空</button>
    <button @click="copy">复制</button>
    <br/>
    <br/>
    <span title="字符太多可缩小看">预览放大/缩小：</span><input v-model="zoom" type="range" min="1" max="10">{{zoom}}
    <hr/>
    <div :style="'zoom:' + zoom / 10">
      <div v-html="innerHTML"></div>
    </div>
    <div ref="filter" style="position: absolute; right: 0;top: 0"></div>
  </div>
</template>

<script>

export default {
  data () {
    return {
      image: null,
      ascii_x_dots: 2,
      ascii_y_dots: 4,
      threshold: 127,
      ascii_width: 50,
      ascii_height: null,
      innerHTML: '',
      text: '',
      zoom: 10
    }
  },
  mounted () {
    const image = document.createElement('img')
    image.src = require('@/assets/logo.png')
    image.onload = () => {
      this.imgOnload()
    }
    this.image = image
  },
  methods: {
    getImg(_) {
      const files = _.target.files
      if (files[0]){
        const file = files[0]
        this.image.src = this.getObjectURL(file)
      }
    },
    getObjectURL(file) {
        var url = null ;
        // 下面函数执行的效果是一样的，只是需要针对不同的浏览器执行不同的 js 函数而已
        if (window.createObjectURL!=undefined) { // basic
            url = window.createObjectURL(file) ;
        } else if (window.URL!=undefined) { // mozilla(firefox)
            url = window.URL.createObjectURL(file) ;
        } else if (window.webkitURL!=undefined) { // webkit or chrome
            url = window.webkitURL.createObjectURL(file) ;
        }
        return url ;
    },
    imgOnload() {
      const {
        ascii_width,
        ascii_x_dots,
        ascii_y_dots,
        image
      } = this
      this.ascii_height = Math.ceil(ascii_width * ascii_x_dots * (image.height / image.width) / ascii_y_dots);
      const ascii_height = this.ascii_height
      const canvas = document.createElement('canvas')
      canvas.width = ascii_width * ascii_x_dots;
      canvas.height = ascii_height * ascii_y_dots;
      const ctx = canvas.getContext('2d')
      ctx.drawImage(image, 0, 0, canvas.width, canvas.height)
      let ascii = [];
      for (let y = 0; y < canvas.height; y += ascii_y_dots) {
        let line = '';
        for (let x = 0; x < canvas.width; x += ascii_x_dots) {
            let char = this.ImageData2Braille(ctx.getImageData(x, y, ascii_x_dots, ascii_y_dots));
            line += char
        }
        ascii.push(line);
      }
      this.text = ascii.join('\n');
      this.innerHTML = ascii.join('<br/>');
      //********************黑白滤镜****************************
      // const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height)
      // const data = imgData.data
      // for (let i = 0;i < imgData.width * imgData.height;i++){
      //   let R = data[i * 4 + 0];
      //   let G = data[i * 4 + 1];
      //   let B = data[i * 4 + 2];
      //   let grey = R * 0.3 + G * 0.59 + B * 0.11;
      //   if (grey > 125){
      //     grey=255;
      //   } else { 
      //     grey = 0;
      //   } 
      //   imgData.data[i * 4 + 0] = grey;
      //   imgData.data[i * 4 + 1] = grey;
      //   imgData.data[i * 4 + 2] = grey;
      // }
      // ctx.putImageData(imgData, 0, 0)
      // this.$refs.filter.innerHTML = ''
      // this.$refs.filter.appendChild(canvas)
      //************************************************
    },
    ImageData2Braille(data) {
      const {
        ascii_x_dots,
        ascii_y_dots,
        threshold
      } = this
      if (data.width != ascii_x_dots || data.height != ascii_y_dots) {
        throw TypeError("Braille image data must be " + ascii_x_dots + "px wide and " + ascii_y_dots + "px high.");
      }
      let dots = [];
      for (let i = 0; i < ascii_x_dots * ascii_y_dots; i++) {
        dots[i] = data.data.subarray(i * 4, (i + 1) * 4);
      }
      // Reorder dots to match Braille dot order
      dots = [dots[0], dots[2], dots[4], dots[1], dots[3], dots[5], dots[6], dots[7]];
      dots = dots.map(function (rgba_array) {
        return (rgba_array[0] + rgba_array[1] + rgba_array[2]) / 3;
      }).map(function (grey) {
        return +(grey < threshold);
      });
      // Braille Unicode range starts at U2800 (= 10240 decimal)
      // Each of the eight dots is mapped to a bit in a byte which
      // determines its position in the range.
      // https://en.wikipedia.org/wiki/Braille_Patterns
      return String.fromCharCode(10240 + parseInt(dots.reverse().join(''), 2));
    },
    copy () {
      const input = document.createElement("input");
      input.value = this.text
      input.select(); // 选中文本
      document.execCommand("copy"); // 执行浏览器复制命令
      alert("复制成功");
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 40px;
}
button + button {
  margin-left: 20px;
}
</style>
