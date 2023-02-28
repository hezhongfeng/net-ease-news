<template>
  <div class="picture-in-picture">
    <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight">
      画中画
    </canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import Cover from "../../assets/images/cover_v2.jpg";
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

const imageDatas = [
  {
    src: Cover,
    width: "750",
    height: "1206",
  },
  {
    src: P1,
    width: "1875",
    height: "3015",
    innerImg: {
      width: "152",
      height: "244",
      left: "370",
      top: "1068",
    },
  },
  {
    src: P2,
    width: "1875",
    height: "3015",
    innerImg: {
      width: "556",
      height: "894",
      left: "1251",
      top: "1050",
    },
  },
];

// let radio = 556 / 1875;
let radio = 1;
// let radio = 0.2965333;

let timer = null;

const loadImages = async () => {
  const promises = [];
  for (const imageData of imageDatas) {
    promises.push(
      new Promise((resolve, reject) => {
        const img = new Image();
        img.src = imageData.src;
        img.onload = () => {
          resolve();
        };
        img.onerror = () => {
          reject();
        };
        imageData.img = img;
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

  const currentImage = imageDatas[0];

  const nextImage = imageDatas[1];

  // // 重复计算
  // const temp1 =
  //   (nextImage.width - nextImage.innerImg.width / radio) /
  //   (nextImage.width - nextImage.innerImg.width);

  // // 由局部小窗口在整个屏幕，缩小到整个图片都在屏幕
  // ctx.drawImage(
  //   nextImage.img,
  //   nextImage.innerImg.left * temp1, // 最初是innerLeft，最后是0 AN/AT = AE/AI=AQ/AR
  //   nextImage.innerImg.top * temp1, // EN/IT= AE/AI=AQ/AR
  //   nextImage.innerImg.width / radio, //   开始是innerWidth，慢慢变大到imageOriginalWith
  //   nextImage.innerImg.height / radio,
  //   0,
  //   0,
  //   canvasWidth,
  //   canvasHeight
  // );

  // 由整个图片都在屏幕，缩小到下一个图片的小窗口
  const asau =
    (currentImage.width - currentImage.width * radio) /
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
    canvasWidth * radio,
    canvasHeight * radio
  );

  radio = radio * 0.993;
};
</script>

<style lang="scss">
.picture-in-picture {
  width: 100%;
}
</style>
