<template>
  <div class="home">
    <div class="btnwrap">
      <div @click="changeBrushColor()" class="brushColor" ref="brushColor">
        <chrome-picker class="colorPicker" :value="colors" @input="updateValue" v-if="showColorPicker"></chrome-picker>
      </div>
      <div class="brushWidth">
         <label >画笔线条粗细:</label>
        <input type="range" name="vol" min="1" max="100" v-model="width">
      </div>
      <div class="brushWidth">
         <label >字体大小:</label>
        <input type="range" name="vol" min="18" max="50" v-model="fontSize">
      </div>
      <button @click="freeDraw()" class="freeDrawBtn">自由绘制</button>
      <button @click="lineDraw()" class="lineDrawBtn">直线</button>
      <button @click="circleDraw()" class="cricleDrawBtn">圆</button>
      <button @click="rectDraw()" class="rectDrawBtn">矩形</button>
      <button @click="triangleDraw()" class="triangleDrawBtn">三角形</button>
      <button @click="textDraw()" class="textDrawBtn">文本</button>
      <button @click="imgDraw()" class="imgDrawBtn">本地图片</button>
      <input type="file" accept="image/*" style="display:none" id="uploadfile" @change="uploadFile" />
      
      <button @click="clear()" class="clearBtn">清除</button>
      <button @click="selectDraw()" class="selectDrawBtn">可选中</button>
      <button @click="unselectDraw()" class="selectDrawBtn">不可选中</button>
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
      fontSize:18,
      showColorPicker:false,
      canvas:null,
      drawingObject:null,
      textObject:null,
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
    },
    lineDraw(){
      this.currentType = 'line';
      this.ininLine();
    },
    circleDraw(){
      this.currentType = 'circle';
      this.initCircle();
    },
    rectDraw(){
      this.currentType = 'rect';
      this.initRect();
    },
    triangleDraw(){
      this.currentType = 'triangle';
      this.initTriangle();
    },
    textDraw(){
      this.currentType = 'text';
    },
    imgDraw(){
      document.getElementById("uploadfile").click();
    },
    uploadFile(e){
      this.canvas.isDrawingMode = false;
      this.removeTextObject();
      var file = e.target.files[0];
      var reader = new FileReader();
      reader.onload = (e) => {
        var data = e.target.result;                    
        fabric.Image.fromURL(data, (img) => { 
          this.canvas.add(img).renderAll();
        });
      };
      reader.readAsDataURL(file);
      e.target.value='';
    },

    toggleDrawingObject(canvasObject){
      this.canvas.isDrawingMode = false;
      this.canvas.selection = false;
      canvasObject.selectable = false;
      if (this.drawingObject) {
        this.canvas.remove(this.drawingObject);
      }
      this.canvas.add(canvasObject);
      this.drawingObject = canvasObject;

      if (this.textObject) {
        this.textObject.exitEditing();
        this.textObject = null;
      }

    },
    ininLine(){
      let canvasObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
        fill: this.colors,
        stroke: this.colors,
        strokeWidth: this.strokeWidth
      });
      this.toggleDrawingObject(canvasObject);
    },
    initCircle(){
      let left = this.mouseFrom.x;
      let top = this.mouseFrom.y;
      let radius = Math.sqrt((this.mouseTo.x - left) * (this.mouseTo.x - left) + (this.mouseTo.y - top) * (this.mouseTo.y - top)) / 2;
      let  canvasObject = new fabric.Circle({
          left: left,
          top: top,
          stroke: this.colors,
          fill: "rgba(255, 255, 255, 0)",
          radius: radius,
          strokeWidth: this.strokeWidth
        });
      this.toggleDrawingObject(canvasObject);
    },
    initRect(){
      let left = this.mouseFrom.x;
      let top = this.mouseFrom.y;
      let width = this.mouseTo.x - this.mouseFrom.x;
      let height = this.mouseTo.y - this.mouseFrom.y;
      let  canvasObject = new fabric.Rect({
          left: left,
          top: top,
          width : width, 
          height : height,
          stroke: this.colors,
          fill: "rgba(255, 255, 255, 0)",
          strokeWidth: this.strokeWidth
        });
      this.toggleDrawingObject(canvasObject);
    },
    initTriangle(){
      let left = this.mouseFrom.x;
      let top = this.mouseFrom.y;
      let height = this.mouseTo.y - this.mouseFrom.y;
      let width = Math.sqrt(Math.pow(height, 2) + Math.pow(height / 2.0, 2));
      let  canvasObject = new fabric.Triangle({
          left: left,
          top: top,
          width : width, 
          height : height,
          stroke: this.colors,
          fill: "rgba(255, 255, 255, 0)",
          strokeWidth: this.strokeWidth
        });
      this.toggleDrawingObject(canvasObject);
    },
   initText(){
      this.canvas.isDrawingMode = false;
      this.canvas.selection = false;
      this.textObject = new fabric.Textbox("", {
          left: this.mouseFrom.x,
          top: this.mouseFrom.y,
          fontSize: this.fontSize,
          fill: this.colors,
          hasControls: false
        });
      this.canvas.add(this.textObject);
      this.textObject.enterEditing();
      this.textObject.hiddenTextarea.focus();
    },
    initEvent(){
      let eventType = ['line','circle','rect','triangle','text'];
      this.canvas.on('mouse:down', (options) => {
        if(eventType.indexOf(this.currentType) != -1){
          this.mouseFrom.x = options.e.clientX;
          this.mouseFrom.y = options.e.clientY - 50;
          this.idDrawing = true;
          switch(this.currentType){
            case 'text':
              this.initText();
              break;
            default:
              break;
          }
        }
      });
      this.canvas.on('mouse:move', (options) => {
        if(this.idDrawing && eventType.indexOf(this.currentType) != -1){
          this.mouseTo.x = options.e.clientX;
          this.mouseTo.y = options.e.clientY - 50;
          switch(this.currentType){
            case 'line':
              this.ininLine();
              break;
            case 'circle':
              this.initCircle();
              break;
            case 'rect':
              this.initRect();
              break;
            case 'triangle':
              this.initTriangle();
              break;
            
            default:
              break;
          }
        }
      });
      this.canvas.on("mouse:up", (options) => {
        if(eventType.indexOf(this.currentType) != -1){
          this.mouseTo.x = options.e.clientX;
          this.mouseTo.y = options.e.clientY - 50;
          this.drawingObject = null;
          this.idDrawing = false;
          this.resetMove();
        }
      }); 
       
    },
    resetMove(){
      this.mouseFrom = {};
      this.mouseTo = {};      
    },
    removeTextObject(){
      this.currentType = '';
      if (this.textObject) {
        this.textObject.exitEditing();
        this.textObject = null;
      }
    },
    clear(){
      this.canvas.clear();
      this.canvas.backgroundColor = '#efefef';
      this.resetMove();
    },
    selectDraw(){
      this.removeTextObject();
      this.canvas.getObjects().map(item =>{
        item.set('selectable', true)
      });
    },
    unselectDraw(){
      this.removeTextObject();
      this.canvas.getObjects().map(item =>{
        item.set('selectable', false)
      });
    },
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
  
  .freeDrawBtn,.lineDrawBtn,.cricleDrawBtn,.rectDrawBtn,
  .triangleDrawBtn,.textDrawBtn,.clearBtn,
  .selectDrawBtn,.imgDrawBtn{
    margin-left: 10px;
    &:hover{
      cursor: pointer;
    }
  }

}


</style>
