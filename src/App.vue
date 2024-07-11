<template>
  <div id="nav_header" class="nav-header" :class="{ fixed: isHeaderFixed }">
    <div class="btn-group">
      <div class="btn" :id="`btn_mode_relative`" @click="mode = 'relative'" :class="{ active: mode === 'relative' }">
        照片墙（relative版）
      </div>
      <div class="btn" :id="`btn_mode_absolute`" @click="mode = 'absolute'" :class="{ active: mode === 'absolute' }">
        照片墙（absolute版）
      </div>
      <!-- <div class="btn" :id="`btn_mode_filter`" @click="mode = 'filter'" :class="{ active: mode === 'filter' }">
        照片墙（点不到版）
      </div> -->
      <div class="btn" :id="`btn_mode_relative-anime`" @click="mode = 'relative-anime'"
        :class="{ active: mode === 'relative-anime' }">
        照片墙（relative-anime版）
      </div>
      <div class="btn" :id="`btn_mode_absolute-anime`" @click="mode = 'absolute-anime'"
        :class="{ active: mode === 'absolute-anime' }">
        照片墙（absolute-anime版）
      </div>
      <!-- <div class="btn" :id="`btn_mode_button`" @click="mode = 'button'" :class="{ active: mode === 'button' }">
        按钮特效展示
      </div> -->
      <div id="nav_mask" :style="{
        left: mask.left + 'px',
        width: mask.width + 'px',
      }"></div>
    </div>

    <!-- <button class="togg-btn" @click="toggleTheme">切换主题</button> -->

    <div class="togg-btn">
      <sun-moon v-model="isDark" :delay="viewTransitionDuration" @change="toggleTheme"></sun-moon>
    </div>
  </div>

  <div class="content" :class="mode">
    <photo-effect-relative v-if="mode === 'relative'" />
    <photo-effect v-if="mode === 'absolute'" />
    <photo-effect-relative-filter v-if="mode === 'filter'" />
    <photo-effect-relative-anime v-if="mode === 'relative-anime'" />
    <photo-effect-anime v-if="mode === 'absolute-anime'" />
    <button-effect v-if="mode === 'button'" />
  </div>
</template>

<script setup lang="ts">
import anime from 'animejs/lib/anime.es.js'
import { ref, onMounted, reactive, watch } from 'vue';
import photoEffectRelativeFilter from './views/photo-effect-relative-filter.vue'
import photoEffectRelative from './views/photo-effect-relative.vue'
import photoEffect from './views/photo-effect.vue'
import photoEffectRelativeAnime from './views/photo-effect-relative-anime.vue'
import photoEffectAnime from './views/photo-effect-anime.vue'
import buttonEffect from "./views/button-effect.vue"

import sunMoon from './components/sun-moon.vue';


const mode = ref('relative')
const isHeaderFixed = ref(false)

const mask = reactive({
  left: 0,
  width: 0,
})

watch(mode, (val: string) => {
  setMaskInfo(val)
})

function setMaskInfo(mode: string) {
  const modeBtnInfo = document.getElementById(`btn_mode_${mode}`)?.getBoundingClientRect()
  // console.log(modeBtnInfo)
  if (!modeBtnInfo) return
  // mask.left = modeBtnInfo.left
  // mask.width = modeBtnInfo.width

  anime({
    targets: '#nav_mask',
    left: modeBtnInfo.left,
    width: modeBtnInfo.width,
    duration: 2000,
    complete: function () {
      setTimeout(() => {
        mask.left = modeBtnInfo.left
      mask.width = modeBtnInfo.width
      }, 100)
    }
  });
}

const isDark = ref(false)
  const viewTransitionDuration = 700

const toggleTheme = (event: MouseEvent) => {
  const x = event.clientX;
  const y = event.clientY;
  const endRadius = Math.hypot(
    Math.max(x, innerWidth - x),
    Math.max(y, innerHeight - y)
  );

  function transition_func() {
    const root = document.documentElement;
    isDark.value = root.classList.contains("dark");
    root.classList.remove(isDark.value ? "dark" : "light");
    root.classList.add(isDark.value ? "light" : "dark");
  }

  // let isDark: boolean;
  if (!document.startViewTransition) {
    transition_func();
    return;
  }

  // @ts-ignore
  const transition = document.startViewTransition(transition_func);

  transition.ready.then(() => {
    const clipPath = [
      `circle(0px at ${x}px ${y}px)`,
      `circle(${endRadius}px at ${x}px ${y}px)`,
    ];
    document.documentElement.animate(
      {
        clipPath: isDark.value ? clipPath.reverse() : clipPath,
      },
      {
        duration: viewTransitionDuration,
        easing: "ease-in-out",
        pseudoElement: isDark.value ? "::view-transition-old(root)" : "::view-transition-new(root)",
      }
    );
  });
};


onMounted(() => {
  setMaskInfo(mode.value)

  const nav_header = document.getElementById("nav_header")
  const headerInfo = nav_header?.getBoundingClientRect()
  if (!headerInfo) return

  // document.getElementById("app")?.addEventListener("scroll", (ev: Event) => {
  //   console.log((ev.target as HTMLElement)?.scrollTop)
  //   if (headerInfo.bottom < (ev.target as HTMLElement)?.scrollTop) {
  //     isHeaderFixed.value = true
  //   }
  // })
})
</script>


<style scoped>
.nav-header {
  height: 50px;
  position: relative;
  background: #000;
  padding-right: 70px;
}

.nav-header .btn-group {
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
}

.nav-header .togg-btn {
  background: transparent;
  position: absolute;
  top: 5.5px;
  right: 25.5px;
  border-radius: 50%;
  width: 39px;
  height: 39px;
}


.nav-header.fixed {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 999;
}

#nav_mask {
  width: 200px;
  height: 50px;
  top: 0;
  position: absolute;
  border-radius: 22px 22px 0 0;
  z-index: 12;
  background: #fff;
  /* box-shadow: 5px 5px 37px rgba(0, 0, 0, 0.58); */
}

#nav_mask::before {
  content: "";
  position: absolute;
  bottom: 0;
  left: -20px;
  width: 20px;
  height: 20px;
  /* background: radial-gradient(circle at 0 0, transparent 0, transparent 20px, #fff 21px, #fff); */
  background: #fff;
  clip-path: path('M 20 0 A 20 20 0 0 1 0 20 H 20 z');
  -webkit-clip-path: path('M 20 0 A 20 20 0 0 1 0 20 H 21 V 0 z');
}

#nav_mask::after {
  content: "";
  position: absolute;
  bottom: 0;
  right: -20px;
  width: 20px;
  height: 20px;
  /* background: radial-gradient(circle at 100% 0, transparent 0, transparent 20px, #fff 21px, #fff); */
  background: #fff;
  clip-path: path('M 0 0 A 20 20 0 0 0 20 20 H 0 z');
  -webkit-clip-path: path('M 0 0 A 20 20 0 0 0 20 20 H -1 V 0 z');
}


.nav-header .btn-group>.btn {
  border: none;
  outline: none;
  height: 50px;
  line-height: 50px;
  cursor: pointer;
  padding: 0 20px;
  position: relative;
  mix-blend-mode: difference;
  z-index: 13;
}

.nav-header .btn-group>.btn.active {}




.content {
  width: 100%;
  padding: 100px;
  box-sizing: border-box;
}

.content.absolute, .content.absolute-anime {
  height: calc(100% - 50px);
  overflow: hidden;
}

.content.relative {
  overflow: hidden;
}
</style>
