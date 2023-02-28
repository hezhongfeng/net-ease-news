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
const { imagesData } = defineProps(["imagesData"]);

const canvas = ref();

let ctx = null;

// 最初的缩小系数
let factor = 1;

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

onMounted(async () => {
  ctx = canvas.value.getContext("2d");
  await loadImages();
  // 画布和图片准备好了，开始画图
  start();
});

const canvasWidth = document.documentElement.clientWidth;

// 按照图片的长宽比，计算出canvas的高度，注意不能使用样式去控制canvas
const canvasHeight = canvasWidth / 0.6218;

const start = () => {
  step();
};

const step = () => {
  // 缩小到PIP的大小和位置
  if (factor < 152 / 1875) {
    window.cancelAnimationFrame(requestId);
  } else {
    draw();
    requestId = window.requestAnimationFrame(step);
  }
  // draw();
};

const draw = () => {
  const currentImage = imagesData[0];

  const nextImage = imagesData[1];

  // 重复计算
  const temp1 =
    (nextImage.width - nextImage.innerImg.width / factor) /
    (nextImage.width - nextImage.innerImg.width);

  // 由局部小窗口在整个屏幕，缩小到整个图片都在屏幕
  ctx.drawImage(
    nextImage.img,
    nextImage.innerImg.left * temp1, // 最初是innerLeft，最后是0 AN/AT = AE/AI=AQ/AR
    nextImage.innerImg.top * temp1, // EN/IT= AE/AI=AQ/AR
    nextImage.innerImg.width / factor, //   开始是innerWidth，慢慢变大到imageOriginalWith
    nextImage.innerImg.height / factor,
    0,
    0,
    canvasWidth,
    canvasHeight
  );

  // 由整个图片都在屏幕，缩小到下一个图片的小窗口
  const asau =
    (currentImage.width - currentImage.width * factor) /
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
    canvasWidth * factor,
    canvasHeight * factor
  );

  factor = factor * 0.993;
};
</script>

<style lang="scss">
.picture-in-picture {
  width: 100%;
}
</style>
