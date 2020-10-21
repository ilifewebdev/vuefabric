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
      <button @click="freeDraw()" class="freeDrawBtn">自由绘制</button>
      <button @click="lineDraw()" class="lineDrawBtn">直线</button>
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
      width:1,
      strokeWidth:1,
      showColorPicker:false,
      canvas:null,
      drawingObject:null,
      mouseFrom: {},
      mouseTo: {},
      currentType:'free',
      idDrawing:false,
    }
  },
  watch: {
    width: function (val) {
      console.log(val);
      this.canvas.freeDrawingBrush.width = parseInt(this.width, 10);
      this.strokeWidth = parseInt(this.width, 10);
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
    initfreeDraw(){
      if(this.canvas == null){
        this.canvas = new fabric.Canvas("c");
        this.canvas.backgroundColor = '#efefef';
        this.canvas.isDrawingMode= 1;
      }
      this.canvas.freeDrawingBrush.color = this.colors;
      this.canvas.freeDrawingBrush.width = this.width;
      this.canvas.renderAll();
    },
    freeDraw(){
      this.canvas.isDrawingMode = true;
      this.currentType = 'free';
      this.resetLine();
    },
    lineDraw(){
      this.currentType = 'line';
      this.ininLine();
      this.resetLine();
    },
    ininLine(){
      this.canvas.isDrawingMode = false;
      this.canvas.selection = false;
      let canvasObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
        fill: this.colors,
        stroke: this.colors,
        strokeWidth: this.strokeWidth
      });
      canvasObject.selectable = false;
      if (this.drawingObject) {
        this.canvas.remove(this.drawingObject);
      }
      this.canvas.add(canvasObject);
      this.drawingObject = canvasObject;
    },
    initEvent(){
      this.canvas.on('mouse:down', (options) => {
        if(this.currentType == 'line'){
          this.mouseFrom.x = options.e.clientX;
          this.mouseFrom.y = options.e.clientY - 50;
          this.idDrawing = true;
        }
      });
      this.canvas.on('mouse:move', (options) => {
        if(this.idDrawing && this.currentType == 'line'){
          this.mouseTo.x = options.e.clientX;
          this.mouseTo.y = options.e.clientY - 50;
          this.ininLine();
        }
      });
      this.canvas.on("mouse:up", (options) => {
        if(this.currentType == 'line'){
          this.mouseTo.x = options.e.clientX;
          this.mouseTo.y = options.e.clientY - 50;
          this.drawingObject = null;
          this.idDrawing = false;
        }
      }); 
       
    },
    resetLine(){
      this.mouseFrom = {};
      this.mouseTo = {};
    },
    clear(){
      this.canvas.clear();
      this.canvas.backgroundColor = '#efefef';
      this.resetLine();
    }
  },
  mounted(){
    this.initBrushColor();
    this.initfreeDraw();
    this.initEvent();
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
  
  .freeDrawBtn,.lineDrawBtn,.clearBtn{
    margin-left: 10px;
    &:hover{
      cursor: pointer;
    }
  }

}


</style>
