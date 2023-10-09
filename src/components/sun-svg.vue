<template>
  <svg :width="width" :height="height" viewBox="0 0 200 200" style="overflow: hidden; border-radius: 50%;" >
    <circle :cx="info.center[0]" :cy="info.center[1]" :r="info.radius" :stroke="color" :stroke-width="info.strokeWidth"
      fill="none">
    </circle>
    <circle v-for="(item, index) in info.other_circles" class="ani-circles" :key="index" :cx="item.cx" :cy="item.cy"
      :r="item.radius" :fill="color"></circle>
  </svg>
</template>

<script lang="ts" setup>
import { reactive,watch,onMounted } from 'vue';

const props = defineProps<{
  color: string,
  width: string,
  height: string,
  backgroundColor: string,
}>()

const other_circles_radius = 8
const info = reactive<any>({
  radius: 35,
  center: [100, 100],
  strokeWidth: 10,

  other_circles: []
})

const math_res: any = {}
function get_math_res(deg: number, type: 'cos' | 'sin') {
  if (math_res[type + deg]) return math_res[type + deg]
  return (math_res[type + deg] = Math[type](Math.PI * 2 * (deg / 360)))
}

function get_other_circles(distance: number) {
  const start_distance = info.radius + other_circles_radius / 2
  const end_distance = start_distance + distance
  // const circle_infos = []
  for (let i = 0; i < 360; i = i + 45) {
    const cx = get_math_res(i, 'cos') * end_distance + info.center[0]
    const cy = get_math_res(i, 'sin') * end_distance + info.center[1]
    // circle_infos.push({
    //   cx,
    //   cy,
    //   radius: other_circles_radius,
    //   fill: 'black'
    // })
    info.other_circles[i / 45] = {
      cx,
      cy,
      radius: other_circles_radius,
      fill: 'black'
    }

  }
}



let requestFrameId: number, distance: number = 20
get_other_circles(distance)



</script>
