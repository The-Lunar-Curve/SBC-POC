<template>
  <v-container>
	<table>
		<tbody>
			<tr>
				<td>
					<canvas id="canvas" width="1000" height="1000"></canvas>
				</td>
			</tr>
		</tbody>
	</table>
  <v-row class="d-flex align-start justify-center">
    <v-col class="d-flex" cols="4">
      <v-btn color="black" dark x-large @click="printAll">
        Find
      </v-btn>
    </v-col> 
    <v-col cols="4" class="d-flex align-center justify-center">
      <v-text-field
      label="Number of colors"
      v-model="colors"
      outlined
      type="number"
      width="20px"
      >

      </v-text-field>
    </v-col>
    <v-col cols="4" class="d-flex align-center justify-center">
      <v-text-field
      label="tolernance"
      v-model="tolerance"
      outlined
      type="number"
      width="20px"
      >

      </v-text-field>
    </v-col>
  </v-row>

  <v-row class="mt-8">
    <v-col cols="2" class="d-flex align-items" v-for="(item, index) in outputs" :key="index">
      <div class="prime ml-2" :style="{backgroundColor: `rgb(${item.rgb})`}">
        {{item.value}}
      </div>
    </v-col>
  </v-row>
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
      colors: 5,
      tolerance: 20,
      clusteredColors: []
    }
  },
  mounted(){
    this.img.crossOrigin = 'anonymous';
    this.img.src = `https://live.staticflickr.com/${this.images[0]}`;

      this.ctx = this.canvas.getContext('2d');
      this.ctx.drawImage(this.img, 0, 0);
      this.img.style.display = 'none';
  },
  computed: {
    canvas(){
      return document.getElementById('canvas');
    },
    storedValues() {
      let colors = [];
      for (var i = 0; i < this.outputs.length; i++) {
        let value = this.outputs[i].key
        colors.push(value);
      } 
      return colors
    }
  },
  methods: {
    pick(event, destination, ctx){
      var x = event.layerX;
      var y = event.layerY;
      var pixel = ctx.getImageData(x, y, 1, 1);
      var data = pixel.data;

      const rgba = `rgba(${data[0]}, ${data[1]}, ${data[2]}, ${data[3] / 255})`;
      destination.style.background = rgba;
      destination.textContent = rgba;

      return rgba;
    },
    printAll(){
      this.outputs = [];
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

        const isBelowThreshold = ((currentValue) => {
            let redDiff = Math.abs(currentValue.red - element.red)
            let greenDiff = Math.abs(currentValue.green - element.green)
            let blueDiff = Math.abs(currentValue.blue - element.blue)

            if (redDiff < this.tolerance && greenDiff < this.tolerance && blueDiff < this.tolerance) {
              return true
            } else {
              return false
            }
        });

        const ifArrayMatches = (array) => array.every(isBelowThreshold);

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
        this.outputs.push(this.clusteredColors[colorIndex][0])
      }
 
      // find all unique threshold colors ***

      // this.clusteredColors.forEach((element) => {
      //   this.outputs.push(element[0]);
      // })

      }
  }

}
</script>
<style lang="scss" scoped>
    .color-cell {
      color: white;
    }
    .prime {
      width: 100%;
      height: 100%;
      max-width: 100px;
      max-height: 100px;
      min-width: 100px;
      min-height: 100px;
      margin: 5px;
      background-color: #eee;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #ffffff;
    }
</style>