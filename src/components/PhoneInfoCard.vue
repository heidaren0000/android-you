<script>
export default {
  name: 'PhoneInfoCard'
}
</script>
<script setup>
defineProps(['phone-info'])
// to calculate the actuall screensize with resolution and pixel density
// however, it's not used
function scrSizeHelper(scr_res, scr_density) {
  console.log(scr_res, scr_density)
  // split res to height and width number
  let resolutionArray = scr_res.split('x')
  let [width, height] = resolutionArray.map(Number)

  let scr_size = Math.sqrt(width * width + height * height) / scr_density
  return scr_size
}
</script>

<template>
  <mdui-card clickable style="width: 350px; height: wrap">
    <mdui-list>
      <mdui-list-subheader>设备信息</mdui-list-subheader>
      <mdui-list-item>
        android 版本
        <span slot="description">{{ phoneInfo.android_version }}</span>
      </mdui-list-item>
      <mdui-list-item>
        安全补丁版本
        <span slot="description">{{ phoneInfo.sec_patch_version }}</span>
      </mdui-list-item>
      <mdui-divider></mdui-divider>
      <mdui-list-item>
        制造商
        <span slot="description">{{ phoneInfo.vendor }}</span>
      </mdui-list-item>
      <mdui-list-item>
        型号
        <span slot="description">{{ phoneInfo.model }}</span>
      </mdui-list-item>
      <mdui-divider></mdui-divider>
      <mdui-list-item>
        CPU 型号
        <span slot="description">{{ phoneInfo.cpu_model }}</span>
      </mdui-list-item>
      <mdui-list-item>
        指令集
        <span slot="description">{{ phoneInfo.cpu_arch }}</span>
      </mdui-list-item>
      <mdui-divider></mdui-divider>
      <mdui-list-item>
        屏幕分辨率
        <span slot="description">{{ phoneInfo.scr_res }}</span>
      </mdui-list-item>
      <mdui-list-item>
        屏幕 DPI
        <span slot="description">{{ phoneInfo.scr_density }}</span>
      </mdui-list-item>
    </mdui-list>
  </mdui-card>
</template>

<style scoped></style>
