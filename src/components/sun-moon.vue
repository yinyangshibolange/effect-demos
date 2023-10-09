<template>
  <div class="sun-moon" @click="triggerMode" :style="{
    backgroundColor: mode_list && mode_list[mode_index] === 'moon' ? 'black' : 'white'
  }">
    <div class="sun">
      <div v-for="(item, index) in mode_list" :key="index">
        <sun-svg v-if="item === 'sun'" :key="'sun_' + index" color="black" backgroundColor="white" width="39px"
          height="39px" :delay="delay"></sun-svg>
        <moon-svg v-if="item === 'moon'" :key="'moon_' + index" color="white" backgroundColor="black" width="39px"
          height="39px"></moon-svg>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue"
import anime from 'animejs/lib/anime.es.js'
import sunSvg from './sun-svg.vue';
import moonSvg from './moon-svg.vue';

const props = defineProps<{
  modelValue: boolean,
  delay: number,
}>()

const emits = defineEmits(["change", "update:modelValue"])


const mode_list = ref()
const mode_index = ref(0)

onMounted(() => {
  if (!props.modelValue) {
    mode_list.value = ['moon', 'sun']
  } else {
    mode_list.value = ['sun', 'moon']
  }
})

function triggerMode(ev: Event) {
  mode_index.value = mode_index.value + 1
  anime({
    targets: '.sun > div',
    translateY: -100 * mode_index.value + '%',
    duration: 500,
    complete: function () {
      if (mode_list.value[mode_list.value.length - 1] === 'moon') {
        mode_list.value.push('sun')
      } else {
        mode_list.value.push('moon')
      }
      emits("update:modelValue", !props.modelValue)
      emits("change", ev)
    }
  });


}

</script>
<style scoped>
.sun-moon {
  width: 39px;
  height: 39px;
  border-radius: 50%;
  overflow: hidden;
}

.sun {
  width: 100%;
  height: 100%;
  /* background: #000; */
  overflow: hidden;
}

.sun>div {}
</style>