<template>
  <div class="home">
    <div class="btnwrap">
      <div @click="changeBrushColor()" class="brushColor" ref="brushColor">
        <chrome-picker class="colorPicker" :value="colors" @input="updateValue" v-if="showColorPicker"></chrome-picker>
      </div>
      <div class="brushWidth">
         <label >画笔线条粗细:</label>
        <input type="range" name="vol" min="1" max="50" v-model="width">
      </div>
      <button @click="clear()" class="clearBtn">清除</button>
    </div>
    <canvas id="c" width="1440" height="600"></canvas>
  </div>
</template>

<script>
// @ is an alias to /src
import { fabric } from "fabric";
import { Chrome } from 'vue-color';

export default {
  name: 'Home',
  components: {
    'chrome-picker': Chrome,
  },
  data () {
    return {
      colors:'#000000',
      width:10,
      showColorPicker:false,
      canvas:null
    }
  },
  watch: {
    width: function (val) {
      console.log(val);
      this.canvas.freeDrawingBrush.width = parseInt(this.width, 10);
    },
  },
  methods:{
    initBrushColor(){
      this.$refs.brushColor.style.backgroundColor = this.colors;
    },
    updateValue(val){
      console.log(val);
      this.colors = val.hex;
      this.initBrushColor();
      this.canvas.freeDrawingBrush.color = this.colors;
    },
    changeBrushColor(){
      this.showColorPicker = !this.showColorPicker;
    },
    freeDraw(){
      if(this.canvas == null){
        this.canvas = new fabric.Canvas("c");
        this.canvas.backgroundColor = '#efefef';
        this.canvas.isDrawingMode= 1;
      }
      this.canvas.freeDrawingBrush.color = this.colors;
      this.canvas.freeDrawingBrush.width = this.width;
      this.canvas.renderAll();
    },
    clear(){
      this.canvas.clear();
      this.canvas.backgroundColor = '#efefef';
    }
  },
  mounted(){
    this.initBrushColor();
    this.freeDraw();
  }

}
  
</script>

<style lang="scss" scoped>

.btnwrap{
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;

  .brushColor{
    width:40px;
    height: 40px;
    position: relative;
    .colorPicker{
      position: absolute;
      left:0;
      top:40px;
      z-index: 1000;
    }
  }

  .brushWidth{
    margin-left: 10px;
  }
  .clearBtn{
    margin-left: 10px;
    &:hover{
      cursor: pointer;
    }
  }

}


</style>
