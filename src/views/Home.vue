<template>
  <div class="home">
    <canvas id="c" width="400" height="400"></canvas>
  </div>
</template>

<script>
// @ is an alias to /src
import { fabric } from "fabric";

export default {
  name: 'Home',
  mounted(){

    var canvas = new fabric.Canvas("c");
    var rect = new fabric.Rect({
        left: 100,
        top: 100,
        fill: 'red',
        width: 400,
        height: 400
        });
        canvas.add(rect);
    }
  }
  
</script>
