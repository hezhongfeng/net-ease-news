<template>
  <div class="picture-in-picture">
    <canvas ref="canvas"> 画中画 </canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// const props = defineProps();
const canvas = ref();

// canvas的ctx
let ctx = null;

onMounted(() => {
  ctx = canvas.value.getContext("2d");

  console.log(start);
});

const start = () => {
  console.log(ctx);
};
</script>

<style lang="scss">
.picture-in-picture {
}
</style>
