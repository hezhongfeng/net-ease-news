<template>
  <div class="picture-in-picture">
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight">
      画中画
    </canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// eslint-disable-next-line vue/no-setup-props-destructure
const { imagesData, factor } = defineProps(["imagesData", "factor"]);

const canvas = ref();

// 缩放的index
let playIndex = 0;

let ctx = null;

// 最初的变化
let reduce = 1;

// requestAnimationFrame的请求ID，可以用来暂停和取消
let requestId = null;

// 加载所有图片
const loadImages = async () => {
  const promises = [];
  for (const item of imagesData) {
    promises.push(
      new Promise((resolve, reject) => {
        const img = new Image();
        img.src = item.src;
        img.onload = () => {
          resolve();
        };
        img.onerror = () => {
          reject();
        };
        item.img = img;
      })
    );
  }

  await Promise.all(promises);
  console.log("图片加载完毕");
};

// 设备的宽度
const canvasWidth = document.documentElement.clientWidth;

// 按照图片的长宽比，计算出canvas的高度，注意不能使用样式去控制canvas
const canvasHeight = canvasWidth / 0.6218;

let currentImage = null;

let nextImage = null;

const start = () => {
  // 起始的两张图片
  currentImage = imagesData[playIndex];
  nextImage = imagesData[playIndex + 1];
  step();
};

// 切换下一张
const nextStep = () => {
  playIndex += 1;
  if (playIndex > imagesData.length - 2) {
    stop();
  } else {
    currentImage = imagesData[playIndex];
    nextImage = imagesData[playIndex + 1];
    reduce = 1;
    requestId = window.requestAnimationFrame(step);
  }
};

const stop = () => {
  window.cancelAnimationFrame(requestId);
};

const step = () => {
  // 缩小到PIP的大小和位置
  if (reduce < nextImage.innerImg.width / nextImage.width) {
    nextStep();
  } else {
    draw();
    requestId = window.requestAnimationFrame(step);
  }
};

const draw = () => {
  console.log("drawing");
  // 重复计算
  const temp1 =
    (nextImage.width - nextImage.innerImg.width / reduce) /
    (nextImage.width - nextImage.innerImg.width);

  // 由局部小窗口在整个屏幕，缩小到整个图片都在屏幕
  ctx.drawImage(
    nextImage.img,
    nextImage.innerImg.left * temp1, // 最初是innerLeft，最后是0 AN/AT = AE/AI=AQ/AR
    nextImage.innerImg.top * temp1, // EN/IT= AE/AI=AQ/AR
    nextImage.innerImg.width / reduce, //   开始是innerWidth，慢慢变大到imageOriginalWith
    nextImage.innerImg.height / reduce,
    0,
    0,
    canvasWidth,
    canvasHeight
  );

  // 由整个图片都在屏幕，缩小到下一个图片的小窗口
  const asau =
    (currentImage.width - currentImage.width * reduce) /
    (currentImage.width -
      (nextImage.innerImg.width * currentImage.width) / nextImage.width); // AS/AU
  ctx.drawImage(
    currentImage.img,
    0,
    0,
    currentImage.width,
    currentImage.height,
    (nextImage.innerImg.left * asau * canvasWidth) / nextImage.width, // 这里需要换算成实际的canvas的大小
    (nextImage.innerImg.top * asau * canvasHeight) / nextImage.height,
    canvasWidth * reduce,
    canvasHeight * reduce
  );

  reduce = reduce * factor;
};

onMounted(async () => {
  ctx = canvas.value.getContext("2d");
  await loadImages();
  // 画布和图片准备好了，开始画图
  start();
});
</script>

<style lang="scss">
.picture-in-picture {
  width: 100%;
}
</style>
