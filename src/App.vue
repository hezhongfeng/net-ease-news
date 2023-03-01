<template>
  <div class="app">
    <PictureInPicture :images-data="imagesData" :factor="0.992" :play="play" />
    <div class="start" :class="{ playing: play }" ref="touchRef"></div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import { PictureInPicture } from "./components/PictureInPicture";
import { useMousePressed } from "@vueuse/core";

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

const touchRef = ref(null);

const { pressed } = useMousePressed({ target: touchRef });

const play = computed(() => {
  return pressed.value;
});

console.log(play);
</script>

<style lang="scss">
.app {
  min-height: 100vh;
  .start {
    position: absolute;
    left: 45vw;
    bottom: 10vw;
    z-index: 2;
    width: 15vw;
    height: 14vw;
    user-select: none;
    background: url(./assets/images/sprite_v2.png) no-repeat;
    background-position: -41.4vw -79.45vw;
    background-size: 770%;
    animation: start 0.4s steps(1) infinite;
  }
  .playing {
    background-position: -41.4vw -92.8vw;
    animation: none;
  }
  @keyframes start {
    0% {
      background-position: -41.4vw -79.45vw;
    }
    50% {
      background-position: -41.4vw -92.8vw;
    }
    100% {
      background-position: -41.4vw -79.45vw;
    }
  }
}
</style>
