<script setup>
import { computed, onMounted, ref, watch } from 'vue'

let props = defineProps(['screen-cap'])
let screenCap = props.screenCap
let imageUrl = ref(null)
let canvas = document.createElement('canvas')
canvas.width = screenCap.width
canvas.height = screenCap.height

onMounted(() => {
  capRender()
})

watch(screenCap, () => {
  console.log('creating a new image url')
  capRender()
})

function capRender() {
  let context = canvas.getContext('2d')
  let imageData = new ImageData(
    new Uint8ClampedArray(screenCap.data),
    screenCap.width,
    screenCap.height
  )
  context.putImageData(imageData, 0, 0)
  imageUrl.value = canvas.toDataURL()
}

let computedWidth = computed(() => {
  return '' + (canvas.width * 0.3 + 'px')
})
let computedHeight = computed(() => {
  return '' + (canvas.height * 0.3 + 'px')
})
</script>

<template>
  <mdui-card
    @click="printProps"
    clickable
    :style="{
      width: computedWidth,
      height: computedHeight,
      'background-image': 'url(' + imageUrl + ')',
      'background-size': 'cover'
    }"
  >
  </mdui-card>
</template>

<style scoped></style>
