<template>
  <div class="picture-in-picture">
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight">
      画中画
    </canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import Picture from "../../assets/images/cover_v2.jpg";

const canvas = ref();

// canvas的ctx
let ctx = null;

const img = new Image();

onMounted(() => {
  img.src = Picture;
  img.onload = () => {
    start();
    console.log("img.onload");
  };
  ctx = canvas.value.getContext("2d");
});

const start = () => {
  console.log(ctx);

  step();
};

const step = () => {
  draw();
  // window.requestAnimationFrame(step);
};

const canvasWidth = document.documentElement.clientWidth;

// 按照图片的长宽比，计算出canvas的高度，注意不能使用样式去控制canvas
const canvasHeight = canvasWidth / 0.6218;

const draw = () => {
  ctx.drawImage(
    img,
    0,
    0,
    img.width,
    img.height,
    0,
    0,
    canvasWidth,
    canvasHeight
  );
};
</script>

<style lang="scss">
.picture-in-picture {
  width: 100%;
}
</style>
