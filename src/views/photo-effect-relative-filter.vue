
<template>
  <div id="scene" class="photo-container" :style="containerStyle" @mouseover="mousein" @mouseout="mouseout">
    <div v-for="(item, index) in photoList" class="photo-item" :class="{ active: item.active }" :id="'photo_' + index"
      :key="index" :style="{
        ...item,
        width: item.width + 'px',
        height: item.height + 'px',

      }">
      <template v-if="item.active">
        <div class="photo-active" style="padding: 20px;" :style="{
          width: item.active_width + 'px',
          height: item.active_height + 'px',
          top: item.active_top + 'px',
          left: item.active_left + 'px',
        }">
          <video :src="item.video" controls :poster="item.src" :style="{
            width: item.active_width + 'px',
            height: '170px',
          }"></video>
          <!-- <img :src="item.src" :alt="item.alt" style="width: 100%;height: auto;"> -->
          <h2>我是标题</h2>
          <p>2023-6-15</p>
        </div>
      </template>
      <template v-else>
        <img :src="item.src" :alt="item.alt" style="width: 100%;height: 100%;">
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
// 本版本是absolute版，只显示一屏，不能滚动，只能用rows和cols进行初始化
import { ref, onMounted, nextTick } from 'vue'
import type { Ref } from "vue"
defineProps<{ msg: string }>()
const containerStyle = ref({})

interface photoItem {
  width: number,
  height: number,
  top: number | string,
  left: number | string,
  src: string,
  alt: string,
  transform: string,
  zIndex: number,
  active: boolean,
  opacity: number,
  transition: string,
  video: string,
  margin: string,
  active_width: number,
  active_height: number,
  active_left: number,
  active_top: number,
}

// 全局参数
const distance_n = 1.2 // 运动距离系数,越小距离越大
const scale_n = 3 // 缩小系数，越小缩得越小
const opacity_n = 2 // 透明度系数，越小越透明
const gutter = 20 // 图片间隔

const photoList: Ref<photoItem[]> = ref([])

containerStyle.value = {
  padding: `0 ${gutter}px ${gutter}px 0`
}

let containerWidth: number | undefined, containerHeight: number | undefined;
let itemInitWidth = 100 // 图片宽度
let itemInitHeight = 70

const init_radius = Math.sqrt(Math.pow(itemInitWidth, 2) + Math.pow(itemInitHeight, 2))

let math_res: any = {}
let old_photoList: photoItem[];

function getSinRes(x: number, y: number, type: 'sin' | 'cos') {
  if (math_res[`${type}_` + x + "_" + y]) {
    return math_res[`${type}_` + x + "_" + y]
  }
  let res;
  if (x == 0) {
    res = Math[type](Math.atan(1))
  } else {
    res = Math[type](Math.atan(y / x))
  }

  math_res[`${type}_` + x + "_" + y] = res
  return res
}

function mousein() {
  document.onmousemove = (move_ev: MouseEvent) => {
    triggerList(2, [move_ev.clientX, move_ev.clientY])
  }
}

function mouseout() {
  document.onmousemove = null
}

function triggerList(n: number, item_center: any[]) {
  photoList.value.forEach((item1: any, index1: number) => {
    const rectInfo = document.getElementById("photo_" + index1)?.getBoundingClientRect()
    if (!rectInfo) return
    // 每个中心点
    const item1_center = [rectInfo.left + rectInfo.width / 2, rectInfo.top + rectInfo.height / 2]

    const distance_x = item1_center[0] - item_center[0]
    const distance_y = item1_center[1] - item_center[1]
    // 计算直线距离
    const radius = Math.sqrt(Math.pow(distance_x, 2) + Math.pow(distance_y, 2))

    // item的放大系数
    const trans_x = (itemInitWidth * n - itemInitWidth) * itemInitWidth / distance_n / radius
    const trans_y = (itemInitHeight * n - itemInitHeight) * itemInitHeight / distance_n / radius


    // 将其他item置地
    item1.zIndex = 2
    // 计算偏移距离
    let _trans_x;
    let _trans_y;
    if (distance_x === 0) {
      _trans_x = 0
    } else {
      _trans_x = Math.abs(trans_x * getSinRes(distance_x, distance_y, 'cos')) * distance_x / Math.abs(distance_x)
    }
    if (distance_y === 0) {
      _trans_y = 0
    } else {
      _trans_y = Math.abs(trans_y * getSinRes(distance_x, distance_y, 'sin')) * distance_y / Math.abs(distance_y)
    }
    item1.transform = `translate(${_trans_x + 'px'}, ${_trans_y + 'px'}) scale(${1 - 1 / radius * init_radius / scale_n})`
    item1.opacity = 1 - 1 / radius * init_radius / opacity_n
  })
}

function init(_photoList: any[]) {
  photoList.value = _photoList
  const rectInfo = document.getElementById("scene")?.getBoundingClientRect()
  containerWidth = rectInfo?.width
  containerHeight = rectInfo?.height
  if (!containerWidth) return
  const cols = Math.floor((containerWidth - gutter) / (itemInitWidth + gutter))
  itemInitHeight = ((containerWidth - gutter) / cols - gutter) / itemInitWidth * itemInitHeight
  itemInitWidth = (containerWidth - gutter) / cols - gutter
  photoList.value.forEach((item: photoItem) => {
    item.width = itemInitWidth
    item.height = itemInitHeight
  })
  old_photoList = JSON.parse(JSON.stringify(photoList.value))
}


onMounted(() => {
  const photoList: photoItem[] = []
  for (let i = 0; i < 223; i++) {
    photoList.push({
      width: itemInitWidth,
      height: itemInitHeight,
      active_width: itemInitWidth,
      active_height: itemInitHeight,
      top: 0,
      left: 0,
      src: "/1.jpg",
      video: "/2.mp4",
      alt: "1.jpg",
      margin: `${gutter}px 0 0 ${gutter}px`,
      transform: "",
      zIndex: 5,
      active: false,
      transition: "all 0.3s ease",
      opacity: 1,
      active_left: 0,
      active_top: 0,
    })
  }
  init(photoList)
})


</script>

<style scoped >
#scene {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  position: relative;
  overflow: hidden;
}

.read-the-docs {
  color: #888;
}


.photo-item {
  /* overflow: hidden; */
  cursor: pointer;
  float: left;
  position: relative;
  transition: all 0.3s ease;

}

.photo-active {
  transition: all 0.3s ease;
  position: absolute;
  /* top: 50%;
  left: 50%;
transform: translate(-50%, -50%); */
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.15);
  z-index: 19;

}

.photo-item.active {
  box-shadow: 0 0 50px 50px rgba(0, 0, 0, 0.2);
}
</style>
