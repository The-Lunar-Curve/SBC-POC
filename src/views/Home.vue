<template>
  <v-container>
  <div class="menu">
    <v-list color="#eee">
      <v-text-field
      label="Number of colors"
      v-model="colors"
      outlined
      type="number"
      >
      </v-text-field>
      <v-text-field
      label="tolerance"
      v-model="tolerance"
      outlined
      type="number"
      >
      </v-text-field>

      <v-text-field
      label="link"
      v-model="link"
      outlined
      type="text"
      >
      </v-text-field>

      <v-btn :loading="isLoading" color="black" dark x-large @click="printAll(tolerance)">
        Find
      </v-btn>
    </v-list>


  </div>
<template v-if="outputs && outputs.length">
<h4 class="mt-8">Exacts</h4>
  <v-row>
    <v-col cols="2" class="d-flex align-items" v-for="(item) in outputs" :key="item.value">
      <div class="prime ml-2" :style="{backgroundColor: `rgb(${item.rgb})`}">
        <div>
        {{item.value}}
        </div>
        <div>
        {{item.rgb}}
        </div>
      </div>
    </v-col>
  </v-row>
</template>

<template v-if="clusteredAverages && clusteredAverages.length">
  <h4 class="mt-8">Averages</h4>
  <v-row>
    <v-col cols="2" class="d-flex align-items" v-for="(item, index) in clusteredAverages" :key="index">
      <div class="prime ml-2" :style="{backgroundColor: `rgb(${item.rgb})`}">
        <div>
        {{item.rgb}}
        </div>
      </div>
    </v-col>
  </v-row>
</template>
	<canvas id="canvas" width="1500" height="1000"></canvas>

  </v-container>
</template>

<script>
import orderBy from 'lodash.orderby';

export default {
  name: 'Home',
  data() {
    return {
      img: new Image(),
      outputs: [],
      images: ['65535/50604677206_d4f1a369da_k.jpg', '1466/25978996034_4d049d33aa_z.jpg'],
      link: 'https://live.staticflickr.com/65535/50604677206_d4f1a369da_k.jpg',
      colors: 5,
      tolerance: 45,
      clusteredColors: [],
      isLoading: false,
      ctx: {}
    }
  },
  watch: {
    link(val){
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

      this.img.src = val;
      this.ctx = {};
      this.ctx = this.canvas.getContext('2d');
      this.ctx.drawImage(this.img, 0, 0);
      this.img.style.display = 'none';
    }
  },
  mounted(){
    this.img.crossOrigin = 'anonymous';
    this.img.src = this.link;

    this.ctx = this.canvas.getContext('2d');
    this.ctx.drawImage(this.img, 0, 0);
    this.img.style.display = 'none';
  },
  computed: {
    canvas(){
      return document.getElementById('canvas');
    },
    clusteredAverages(){
      let clusteredAverages = [];
      this.clusteredColors.forEach((array, index) => {
        var redSum = 0;
        var greenSum = 0;
        var blueSum = 0;

        for(var item in array) {
            redSum += array[item].red;
            greenSum += array[item].green;
            blueSum += array[item].blue;
        }
        
        var count = array.length;
        
        var redAvg = redSum / count;
        var greenAvg = greenSum / count;
        var blueAvg = blueSum / count;
        clusteredAverages[index] = {red: redAvg, green: greenAvg, blue: blueAvg, rgb: `${Math.round(redAvg)}, ${Math.round(greenAvg)}, ${Math.round(blueAvg)}`};
      });
    return clusteredAverages;
    }
  },
  methods: {
    printAll(tolerance){
      this.isLoading = true;

      this.outputs = [];
      this.clusteredColors = [];
      let outputs = [];
      try {
        var imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
        const data = imageData.data;
        var allColors = {};

        // map unique colors
        for (var i = 0; i < data.length; i += 4) {
          var color = `${data[i]}-${data[i + 1]}-${data[i + 2]}`;


          if (allColors[color]) {
            allColors[color].value++
          } else {
            allColors[color] = {
              value: 1,
              rgb: `${data[i]}, ${data[i + 1]}, ${data[i + 2]}`,
              key: color,
              red: data[i],
              green: data[i+1],
              blue: data[i+2]
            }
          }
        }

        let newColorArray = [];
        for ( var newColor in allColors){
          newColorArray.push({
            ...allColors[newColor]
          })
        }

        let sortedColors = orderBy(newColorArray, 'value', 'desc');

        sortedColors.forEach((element) => {

          const ifArrayMatches = (array, index) => {

                let redDiff = Math.abs(this.clusteredAverages[index].red - element.red)
                let greenDiff = Math.abs(this.clusteredAverages[index].green - element.green)
                let blueDiff = Math.abs(this.clusteredAverages[index].blue - element.blue)

                if (redDiff < tolerance && greenDiff < tolerance && blueDiff < tolerance) {
                  return true
                } else {
                  return false
                }
          };
          

          if (this.clusteredColors.length >= this.colors) return;
          
          // clusters exist
          if (this.clusteredColors && this.clusteredColors.length) {


            // find index of array that matches
            
            let matchIndex = this.clusteredColors.findIndex(ifArrayMatches);
            if (matchIndex >= 0) {
              this.clusteredColors[matchIndex].push(element);
            } else {
              this.clusteredColors.push([element]);
            }
          } else {
            this.clusteredColors.push([element]);
          }
          
        });
        
        for (var colorIndex = 0; colorIndex < this.colors; colorIndex++) {
          outputs.push(this.clusteredColors[colorIndex][0])
        }
      } catch (error) {
        console.log(error);
      } finally {
          this.setOutputs(outputs);
          this.isLoading = false;
      }

      },
      setOutputs(payload){
        this.outputs = payload;
      }
  }

}
</script>
<style lang="scss" scoped>
#canvas {
  transform: scale(.7);
}
.menu {
  position: fixed;
  width: 350px;
  height: 100%;
  background-color: #eee;
  left: 0;
  top: 0;
  padding-top: 2em;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  flex-direction: column;
}
    .color-cell {
      color: white;
    }
    .prime {
      width: 100%;
      height: 100%;
      max-width: 120px;
      max-height: 120px;
      min-width: 120px;
      min-height: 120px;
      margin: 5px;
      background-color: #eee;
      border-radius: 50%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: #ffffff;
      border: 1px solid #eee;
    }
</style>