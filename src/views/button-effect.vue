<template>
 <div class="container">
  <div class="tab active">Tab 1</div>

  <div class="text active">text 1</div>
  <div class="text1 active">text 1</div>
  <div class="mask"></div>


  <div class="g-container">
   <div class="g-content">
    <div class="g-filter"></div>
   </div>
  </div>


 </div>
</template>

<script lang="ts" setup>
import { ref } from "vue"
const activeBtn = ref(0)
</script>

<style scoped>
.container {
 background: #fff;
 position: relative;
 min-height: 800px;
}

.tab {
 padding: 10px 20px;
 margin-right: 20px;
 font-size: 24px;
 font-weight: bold;
 color: #000;
 cursor: pointer;
}

.text {
 position: absolute;
 top: 300px;
 left: 330px;
 font-size: 48px;
 font-weight: bold;
 color: #f2f2f2;
 mix-blend-mode: difference;
 transition: opacity 0.5s ease-in-out;
 z-index: 12;
}

.text1 {
 position: absolute;
 top: 305px;
 left: 335px;
 font-size: 48px;
 font-weight: bold;
 color: #333;
 mix-blend-mode: difference;
 transition: opacity 0.5s ease-in-out;
 z-index: 12;
}

/* 
.mask {
 background: #fff;
  position: absolute;
  top: 100px;
  left: 0;
  width: 100px;
  height: 27px;
  z-index: 11;
 
} */

.g-container {
 position: relative;
 width: 300px;
 height: 100px;

 background: #000;
}

.g-container .g-content {
 height: 100px;
 filter: contrast(20);
 background-color: white;
 overflow: hidden;
}

.g-container .g-content .g-filter {
 filter: blur(10px);
 height: 100px;
 background: radial-gradient(circle at 50% -10px, transparent 0, transparent 29px, #000 40px, #000);
}
</style>