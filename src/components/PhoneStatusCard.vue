<script>
export default {
  name: 'PhoneStatusCard'
}
</script>

<script setup>
import { computed } from 'vue'
let status = defineProps(['phone-status'])
let batStat = computed(() => {
  console.log(status.phoneStatus)
  switch (Number(status.phoneStatus.bat_status)) {
    case 2:
      return '正在充电'
    case 3:
      return '正在放电'
    case 4:
      return '已充满电'
    default:
      return '未知'
  }
})
</script>

<template>
  <mdui-card clickable style="width: 350px; height: wrap">
    <mdui-list>
      <mdui-list-subheader>设备状态</mdui-list-subheader>
      <mdui-list-item>
        RAM
        <span slot="description">{{ phoneStatus.ram_total / 1000000 }} GB</span>
      </mdui-list-item>
      <mdui-list-item>
        可用 RAM
        <span slot="description">{{ phoneStatus.ram_available / 1000000 }} GB</span>
      </mdui-list-item>
      <mdui-divider></mdui-divider>
      <mdui-list-item>
        存储空间
        <span slot="description">{{ phoneStatus.storage_total }}</span>
      </mdui-list-item>
      <mdui-list-item>
        可用存储空间
        <span slot="description">{{ phoneStatus.storage_available }}</span>
      </mdui-list-item>
      <mdui-divider></mdui-divider>
      <mdui-list-item>
        运营商
        <span slot="description">{{ phoneStatus.isp }}</span>
      </mdui-list-item>
      <mdui-divider></mdui-divider>
      <mdui-list-item>
        电池电量
        <span slot="description">{{ phoneStatus.bat_level }} %</span>
      </mdui-list-item>
      <mdui-list-item>
        电池状态
        <span slot="description">{{ batStat }}</span>
      </mdui-list-item>
    </mdui-list>
  </mdui-card>
</template>

<style scoped></style>
