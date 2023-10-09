<template>
 <svg :width="width" :height="height" viewBox="0 0 200 200" style="overflow: hidden; border-radius: 50%;">
  <circle  :cx="center[0]" :cy="center[1]" :r="radius" :fill="color">
  </circle>
  <circle  :cx="other_circle.cx" :cy="other_circle.cy" :r="other_circle.radius" :fill="backgroundColor"></circle>
 </svg>
</template>

<script lang="ts" setup>
// import anime from 'animejs/lib/anime.es.js'
const props = defineProps<{
 backgroundColor: string,
 color: string,
 width: string,
 height: string,
}>()

const center = [100, 100]
const radius = 50

const other_circle = {
 cx: 150,
 cy: 60,
 radius: 60,
 fill: "white"
}
</script>
