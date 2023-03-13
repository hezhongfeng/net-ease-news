<template>
  <div class="picture-in-picture">
    <canvas ref="canvas" v-bind:width="canvasWidth" :height="canvasHeight"> </canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const props = defineProps(['imagesData', 'factor', 'play']);

const canvas = ref();

// 缩放的index
let playIndex = 0;

let ctx = null;

// 缩小系数，最初是1
let reduce = 1;

// requestAnimationFrame的请求ID，可以用来暂停和取消
let requestId = null;

// 加载所有图片
const loadImages = async () => {
  const promises = [];
  for (const item of props.imagesData) {
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
  console.log('图片加载完毕');
};

// 设备的宽度
const canvasWidth = document.documentElement.clientWidth;

// 按照图片的长宽比，计算出canvas的高度，注意不能使用样式去控制canvas
const canvasHeight = canvasWidth / 0.6218;

let currentImage = null;

let nextImage = null;

const start = () => {
  // 起始的两张图片
  currentImage = props.imagesData[playIndex];
  nextImage = props.imagesData[playIndex + 1];
  drawCover();
};

console.log(props.play);

watch(
  () => props.play,
  newPlay => {
    if (newPlay) {
      step();
    } else {
      stop();
    }
  },
  {
    immediate: false
  }
);

// 切换下一张
const nextStep = () => {
  playIndex += 1;
  if (playIndex > props.imagesData.length - 2) {
    stop();
  } else {
    currentImage = props.imagesData[playIndex];
    nextImage = props.imagesData[playIndex + 1];
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
  console.log('drawing');

  // 下一张图片
  const nextASAU = (nextImage.width - nextImage.innerImg.width / reduce) / (nextImage.width - nextImage.innerImg.width); // AS/AU

  ctx.drawImage(
    nextImage.img,
    nextImage.innerImg.left * nextASAU, // AT*AS/AU
    nextImage.innerImg.top * nextASAU, // TH*AS/AU
    nextImage.innerImg.width / reduce, // EF
    nextImage.innerImg.height / reduce, // RH
    0,
    0,
    canvasWidth,
    canvasHeight
  );

  // 当前图片，需要后画
  const currentASAU =
    (currentImage.width - currentImage.width * reduce) /
    (currentImage.width - (nextImage.innerImg.width * currentImage.width) / nextImage.width); // AS/AU
  ctx.drawImage(
    currentImage.img,
    0,
    0,
    currentImage.width,
    currentImage.height,
    (nextImage.innerImg.left * currentASAU * canvasWidth) / nextImage.width, // 这里需要换算成实际的canvas的大小
    (nextImage.innerImg.top * currentASAU * canvasHeight) / nextImage.height,
    canvasWidth * reduce,
    canvasHeight * reduce
  );
  reduce = reduce * props.factor;
};

const drawCover = () => {
  // 当前图片
  const currentASAU =
    (currentImage.width - currentImage.width * reduce) /
    (currentImage.width - (nextImage.innerImg.width * currentImage.width) / nextImage.width); // AS/AU
  ctx.drawImage(
    currentImage.img,
    0,
    0,
    currentImage.width,
    currentImage.height,
    (nextImage.innerImg.left * currentASAU * canvasWidth) / nextImage.width, // 这里需要换算成实际的canvas的大小
    (nextImage.innerImg.top * currentASAU * canvasHeight) / nextImage.height,
    canvasWidth * reduce,
    canvasHeight * reduce
  );
};

onMounted(async () => {
  ctx = canvas.value.getContext('2d');
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
