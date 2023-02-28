<template>
  <div class="app">
    <PictureInPicture :images-data="imagesData" :factor="0.99" />
  </div>
</template>

<script setup>
import { PictureInPicture } from "./components/PictureInPicture";

import Cover from "./assets/images/cover_v2.jpg";
import P1 from "./assets/images/p1.jpg";
import P2 from "./assets/images/p2.jpg";
import P3 from "./assets/images/p3.jpg";
import P4 from "./assets/images/p4.jpg";

const imagesData = [
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
  {
    src: P3,
    width: "1875",
    height: "3015",
    innerImg: {
      width: "166",
      height: "267",
      left: "114",
      top: "897",
    },
  },
  {
    src: P4,
    width: "1875",
    height: "3015",
    innerImg: {
      width: "194",
      height: "312",
      left: "85",
      top: "1402",
    },
  },
];
</script>

<style lang="scss">
.app {
  min-height: 100vh;
}
</style>
