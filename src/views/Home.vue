<template>
  <v-container>
	<table>
		<tbody>
			<tr>
				<td>
					<canvas id="canvas" width="700" height="700"></canvas>
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
        {{item.rgb}}
      </div>
    </v-col>
  </v-row>
  </v-container>
</template>

<script>

export default {
  name: 'Home',
  data() {
    return {
      img: new Image(),
      outputs: [],
      images: ['65535/50604677206_d4f1a369da_k.jpg', '1466/25978996034_4d049d33aa_z.jpg'],
      colors: 5,
      tolerance: 100
    }
  },
  mounted(){
    this.img.crossOrigin = 'anonymous';
    this.img.src = `https://live.staticflickr.com/${this.images[1]}`;

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

      // find dominent colors
      for (var index = 0; index < this.colors; index++) {
        let newColorArray = [];

        for ( var newColor in allColors){
          if (this.storedValues.includes(allColors[newColor].key)) {
            continue;
          } else {
            if (this.outputs && this.outputs.length) {

              let redDifference = Math.abs(allColors[newColor].red - this.outputs[this.outputs.length - 1].red);
              let greenDifference = Math.abs(allColors[newColor].green - this.outputs[this.outputs.length - 1].green);
              let blueDifference = Math.abs(allColors[newColor].blue - this.outputs[this.outputs.length - 1].blue);

              if (redDifference > this.tolerance ||  greenDifference > this.tolerance || blueDifference > this.tolerance) {
                newColorArray.push({
                  ...allColors[newColor]
                });
              }
            } else {
              newColorArray.push({
                ...allColors[newColor]
              });
            }
          }
        }

        let maximum = Math.max.apply(Math, newColorArray.map((o) => { return o.value; }))
        let greatestColor = newColorArray.find((o) => { return o.value === maximum; })
        this.outputs.push(greatestColor);
      }

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