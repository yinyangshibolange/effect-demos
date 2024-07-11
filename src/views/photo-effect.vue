
<template>
  <div id="scene" class="photo-container" :style="containerStyle">
    <div v-for="(item, index) in photoList" class="photo-item" :class="{ active: item.active }" :id="'photo_' + index"
      :key="index" :style="{
        ...item,
        width: item.width + 'px',
        height: item.height + 'px',
        top: item.top + 'px',
        left: item.left + 'px',
      }" @click="activeItem(index)" >
      <template v-if="item.active">
        <div style="padding: 20px;">
          <video :src="item.video" controls :poster="item.src" style="width: 100%;height: 170px;"></video>
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
import { ref, onMounted } from 'vue'
import type { Ref } from "vue"
defineProps<{ msg: string }>()
const containerStyle = ref({})

interface photoItem {
  width: number,
  height: number,
  top: number,
  left: number,
  src: string,
  alt: string,
  transform: string,
  zIndex: number,
  active: boolean,
  opacity: number,
  transition: string,
  video: string,
  margin: string,
}

// 全局系数
const distance_n = 1.5 // 运动距离系数,越小距离越大
const scale_n = 5 // 缩小系数，越小缩得越小
const opacity_n = 2 // 透明度系数，越小越透明
const gutter = 20 // 图片间隔

const photoList: Ref<photoItem[]> = ref([])

containerStyle.value = {
  padding: `0 ${gutter}px ${gutter}px 0`
}

let containerWidth: number, containerHeight: number;
let itemInitWidth = 100 // 图片宽度
let itemInitHeight = 60

const init_radius = Math.sqrt(Math.pow(itemInitWidth, 2) + Math.pow(itemInitHeight, 2))

let math_res: any = {}
let old_photoList : photoItem[];

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

function activeItem(index: number) {
  // 先关闭所有激活的item
  unactiveItem()
  const item = photoList.value[index]
  item.active = true

  // 将点击的item放大
  item.width = itemInitWidth * 3
  item.height = itemInitWidth * 3
  // 中心放大
  item.left = item.left - (item.width - itemInitWidth) / 2
  item.top = item.top - (item.height - itemInitHeight) / 2
  if(item.left + gutter < 0) item.left = -gutter
  if(item.top + gutter < 0) item.top = -gutter
  if(item.left + item.width > containerWidth) item.left = containerWidth - item.width
  if(item.top + item.height > containerHeight) item.top = containerHeight - item.height
  item.zIndex = 9

  // item中心点坐标
  const item_center = [item.left + item.width / 2, item.top + item.height / 2]

  triggerList(item, index, item_center)

}

function triggerList(item: any, index: number, item_center: any[]) {
  // item中心点坐标
  // const item_center = [item.left + item.width / 2, item.top + item.height / 2]

  photoList.value.forEach((item1: any, index1: number) => {
    if (index1 === index) return
    // 每个中心点
    const item1_center = [item1.left + item1.width / 2, item1.top + item1.height / 2]

    const distance_x = item1_center[0] - item_center[0]
    const distance_y = item1_center[1] - item_center[1]
    // 计算直线距离
    const radius = Math.sqrt(Math.pow(distance_x, 2) + Math.pow(distance_y, 2))

    // item的放大系数
    const trans_x = (item.width - itemInitWidth) * itemInitWidth / distance_n / radius
    const trans_y = (item.height - itemInitHeight) * itemInitHeight / distance_n / radius


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

function unactiveItem() {
  photoList.value = JSON.parse(JSON.stringify(old_photoList))
}

function init(_photoList: any[], rows: number, cols: number) {
  const rectInfo = document.getElementById("scene")?.getBoundingClientRect()
  const width = rectInfo?.width
  const height = rectInfo?.height
  if(!width || !height) return

  containerWidth = width
  containerHeight = height

  // const cols = Math.floor((width - gutter) / (itemInitWidth + gutter))
  // const rows = Math.floor((height - gutter) / (itemInitHeight + gutter))

  itemInitWidth = (width - gutter) / cols - gutter
  itemInitHeight = (height - gutter) / rows - gutter

  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      photoList.value.push({
        width: itemInitWidth,
        height: itemInitHeight,
        top: i * (itemInitHeight + gutter),
        left: j * (itemInitWidth + gutter),
        src: "http://awfaw.oss-cn-hangzhou.aliyuncs.com/-7MQCQxD1qt98oNsvW7xZL3i.mp4?x-oss-process=video/snapshot,t_0,f_jpg,m_fast",
        video: "http://awfaw.oss-cn-hangzhou.aliyuncs.com/-7MQCQxD1qt98oNsvW7xZL3i.mp4",
        alt: "1.jpg",
        margin: `${gutter}px 0 0 ${gutter}px`,
        transform: "",
        zIndex: 5,
        active: false,
        // transition: "all 0.3s cubic-bezier(.17,.67,.65,1.86)",
        transition: "all 0.3s ease",
        opacity: 1,
      })
    }
  }
  old_photoList = JSON.parse(JSON.stringify(photoList.value))
}


onMounted(() => {
  const photoList: photoItem[] = []

  // row,col
  init(photoList, 5, 8)
})


</script>

<style scoped >
#scene {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  position: relative;
}

.read-the-docs {
  color: #888;
}


.photo-item {
  overflow: hidden;
  cursor: pointer;
  position: absolute;
}

.photo-item.active {
  box-shadow: 0 0 50px 50px rgba(0, 0, 0, 0.2);
}
</style>
