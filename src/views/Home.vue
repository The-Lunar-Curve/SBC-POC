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
        label="threshold"
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
        <v-btn :loading="isLoading" color="black" dark x-large @click="extractColors(tolerance)">
          Find
        </v-btn>
      </v-list>
    </div>
    <div class="content">
      <template v-if="outputs && outputs.length">
          <h4 class="mt-8">Exacts</h4>
        <v-row>
          <v-col cols="2" class="d-flex align-items" v-for="(item) in outputs" :key="item.value">
            <div class="swatch ml-2" :style="{backgroundColor: `rgb(${item.rgb})`}">
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
            <div class="swatch ml-2" :style="{backgroundColor: `rgb(${item.rgb})`}">
              <div>
              {{item.rgb}}
              </div>
            </div>
          </v-col>
        </v-row>
      </template>
      <canvas id="canvas" width="1500" height="1000"></canvas>
    </div>
  </v-container>
</template>

<script>
import { extract } from '@/utils/extract';

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
    clusteredAverages() {
      return extract.findClusterAverages(this.clusteredColors);
    }
  },
  methods: {
    extractColors(tolerance){
      this.isLoading = true;

      this.outputs = [];
      this.clusteredColors = [];
      let outputs = [];
      try {
        var imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);

        let uniqueColors = extract.uniqueColors(imageData.data);
        let sortedColors = extract.sortedColors(uniqueColors);

        sortedColors.forEach((element) => {

          const ifClusterMatches = (array, index) => {
            return extract.clusterMatches(tolerance, index, this.clusteredAverages, element);
          };
          
          if (this.clusteredColors.length >= this.colors) return;
          
          if (this.clusteredColors && this.clusteredColors.length) {
            let clusterIndex = this.clusteredColors.findIndex(ifClusterMatches);

            if (clusterIndex >= 0) {
              this.clusteredColors[clusterIndex].push(element);
            } else {
              this.clusteredColors.push([element]);
            }
            
          } else {
            this.clusteredColors.push([element]);
          }
        });
        
        for (var colorIndex = 0; colorIndex < this.colors; colorIndex++) {
          let combinedColor = this.clusteredColors[colorIndex][0];
          combinedColor.hex = extract.rgbToHexadecimal(combinedColor.red, combinedColor.green, combinedColor.blue);
          outputs.push(combinedColor);
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
.content {
  position: absolute;
  top: 0;
  left: 350px;
  overflow: auto;
  width: calc(100% - 350px);
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.swatch {
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