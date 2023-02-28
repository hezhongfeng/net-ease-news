<template>
  <div class="app">
    <PictureInPicture :images-data="imagesData" />
  </div>
</template>

<script setup>
import { PictureInPicture } from "./components/PictureInPicture";

import Cover from "./assets/images/cover_v2.jpg";
import P1 from "./assets/images/p1.jpg";
import P2 from "./assets/images/p2.jpg";

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
];
</script>

<style lang="scss">
.app {
  min-height: 100vh;
}
</style>
