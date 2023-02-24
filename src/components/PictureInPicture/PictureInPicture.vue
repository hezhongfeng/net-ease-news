<template>
  <div class="picture-in-picture">
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight">
      画中画
    </canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
// import Picture from "../../assets/images/cover_v2.jpg";
import P1 from "../../assets/images/p1.jpg";
import P2 from "../../assets/images/p2.jpg";

const canvas = ref();

// canvas的ctx
let ctx = null;

// 图片最初的尺寸
// const imageOriginalWith = 750;
// const imageOriginalHeight = 1206;

// const imageOriginalWith = 1875;
// const imageOriginalHeight = 3015;

const ImgSrcs = [P1, P2];
const images = [];

// let radio = 556 / 1875;
let radio = 1;

let timer = null;

const loadImages = async () => {
  const promises = [];
  for (const src of ImgSrcs) {
    promises.push(
      new Promise((resolve, reject) => {
        const img = new Image();
        img.src = src;
        img.onload = () => {
          resolve();
        };
        img.onerror = () => {
          reject();
        };
        images.push(img);
      })
    );
  }

  await Promise.all(promises);
  console.log("图片加载完毕");
};

onMounted(async () => {
  await loadImages();

  ctx = canvas.value.getContext("2d");
  start();
});

const start = () => {
  console.log(ctx);

  step();
};

const step = () => {
  // 缩小后停止;
  if (radio < 556 / 1875) {
    console.log(72);
    window.cancelAnimationFrame(timer);
  } else {
    draw();
    timer = window.requestAnimationFrame(step);
  }
  // draw();
};

const canvasWidth = document.documentElement.clientWidth;

// 按照图片的长宽比，计算出canvas的高度，注意不能使用样式去控制canvas
const canvasHeight = canvasWidth / 0.6218;

const draw = () => {
  console.log("drawing");

  // 由大到小
  ctx.drawImage(
    images[1],
    1251 * ((1875 - 556 / radio) / (1875 - 556)), // 最初是innerLeft，最后是0 AN/AT = AE/AI=AQ/AR
    1050 * ((1875 - 556 / radio) / (1875 - 556)), // EN/IT= AE/AI=AQ/AR
    556 / radio, //   开始是innerWidth，慢慢变大到imageOriginalWith
    894 / radio,
    0,
    0,
    canvasWidth,
    canvasHeight
  );

  // width: '556',
  //       height: '894',
  //       left: '1251',
  //       top: '1050',

  // const imageOriginalWith = 1875;
  // const imageOriginalHeight = 3015;

  // ctx.drawImage(
  //   images[1],
  //   0,
  //   0,
  //   1875,
  //   3015,
  //   0,
  //   0,
  //   canvasWidth,
  //   canvasHeight
  // );

  // // 由大到小
  // ctx.drawImage(
  //   images[0],
  //   0,
  //   0,
  //   imageOriginalWith,
  //   imageOriginalHeight,
  //   0,
  //   0,
  //   canvasWidth * radio,
  //   canvasHeight * radio
  // );

  radio = radio * 0.993;
};
</script>

<style lang="scss">
.picture-in-picture {
  width: 100%;
}
</style>
