<script>
export default {
  name: 'MyPhoneView'
}
</script>

<script setup>
import PhoneInfoCard from '@/components/PhoneInfoCard.vue'
import PhoneStatusCard from '@/components/PhoneStatusCard.vue'
import PhoneScrCapCard from '@/components/PhoneScrCapCard.vue'

// adb related
import {
  AdbDaemonWebUsbDeviceManager,
  AdbDaemonWebUsbDeviceWatcher
} from '@yume-chan/adb-daemon-webusb'
import AdbWebCredentialStore from '@yume-chan/adb-credential-web'
import { AdbDaemonTransport } from '@yume-chan/adb'
import { Adb } from '@yume-chan/adb'
import { computed, onBeforeUnmount, reactive, ref } from 'vue'

let connected = ref(false)
let fabHitted = ref(false)
let showErrorDialog = ref(false)
let showUnplugDialog = ref(false)
let adb = null
let phoneStatus = null
let phoneInfo = null
let screenCap = null // will be reactive later

let fabText = computed(() => {
  if (!connected.value && !fabHitted.value) {
    return '连接手机'
  }
  if (!connected.value && fabHitted) {
    return '连接中'
  }
  if (connected.value) {
    return '连接成功!'
  }
  return '未知状态'
})

onBeforeUnmount(() => {
  watcher.dispose()
})

let fabLoading = computed(() => {
  if (!connected.value && fabHitted.value) {
    return true
  }
  return false
})

function deviceChangeHanlder(addedDeviceSerial) {
  if (addedDeviceSerial) {
    return
  } else {
    showUnplugDialog.value = true
  }
}

let watcher = new AdbDaemonWebUsbDeviceWatcher(deviceChangeHanlder, navigator.usb)

async function adbShellRawMode(adb, command) {
  // DANGER! Could lead to hang if it's interactive command
  let result = await adb.subprocess.spawnAndWait(command)
  console.log(result)
  return result.stdout
}

async function connect() {
  fabHitted.value = true
  let device = await requestUsb()
  if (!device) {
    console.log('Enconter error(s) when connecting to device, aborting...')
    return
  }
  let adb = await requestAdb(device)
  if (!adb) {
    console.log('Error: Failed to create adb transport.')
    return
  }
  console.log('connected successfully')
  // get phone info and status
  let phoneInfo = await getPhoneInfo(adb)
  console.log(phoneInfo)
  let phoneStatus = await getPhoneStatus(adb)
  console.log(phoneStatus)
  let screenCap = await getScreenCap(adb)
  return {
    adb,
    phoneInfo,
    phoneStatus,
    screenCap,
    connected: true
  }
}

async function getScreenCap(adb) {
  return await adb.framebuffer()
}

async function getPhoneInfo(adb) {
  console.log(adb)
  return {
    // os version
    android_version: await adb.getProp('ro.build.version.release'),
    sec_patch_version: await adb.getProp('ro.build.version.security_patch'),
    // model and vendor
    model: await adb.getProp('ro.product.model'),
    vendor: await adb.getProp('ro.product.brand'),
    // cpu
    cpu_model: await adbShellRawMode(
      adb,
      `cat /proc/cpuinfo | grep Hardware | sed 's/.*Hardware\t: //'| tr -d '\n'`
    ),
    cpu_arch: await adbShellRawMode(
      adb,
      `cat /proc/cpuinfo | grep Processor | sed 's/.*Processor\t: //'| tr -d '\n'`
    ),
    // screen
    scr_res: await adbShellRawMode(adb, `wm size | sed 's/.*Physical size: //' | tr -d '\n'`),
    scr_density: await adbShellRawMode(
      adb,
      `wm density | sed 's/.*Physical density: //' | tr -d '\n'`
    ),
    scr_size: null
  }
}

async function getPhoneStatus(adb) {
  return {
    // ram
    ram_total: await adbShellRawMode(
      adb,
      `cat /proc/meminfo | grep MemTotal | sed 's/.*MemTotal: //'| tr -d ' kB\n'`
    ),
    ram_available: await adbShellRawMode(
      adb,
      `cat /proc/meminfo | grep MemAvailable | sed 's/.*MemAvailable: //'| tr -d ' kB\n'`
    ),
    // storage
    storage_total: await adbShellRawMode(
      adb,
      `df -h /data | tail -1 | awk '{print $2}' | tr -d '\n'`
    ),
    storage_available: await adbShellRawMode(
      adb,
      `df -h /data | tail -1 | awk '{print $4}' | tr -d '\n'`
    ),
    // isp
    isp: await adb.getProp('gsm.sim.operator.alpha'),
    // battery
    bat_status: await adbShellRawMode(
      adb,
      `dumpsys battery| grep status | sed 's/.*  status: //'| tr -d '\n'`
    ),
    bat_level: await adbShellRawMode(
      adb,
      `dumpsys battery| grep level | sed 's/.*  level: //'| tr -d '\n'`
    )
  }
}

async function requestUsb() {
  const Manager = AdbDaemonWebUsbDeviceManager.BROWSER
  if (!Manager) {
    console.log('Error: WebUSB is not supported in current environment.')
    return
  }
  const device = await Manager.requestDevice()
  if (!device) {
    console.log('Error: No device selected.')
    return
  }
  return device
}

async function requestAdb(device) {
  // connection
  const connection = await device.connect()
  const CredentialStore = new AdbWebCredentialStore()
  const transport = await AdbDaemonTransport.authenticate({
    serial: device.serial,
    connection,
    credentialStore: CredentialStore
  })
  const adb = new Adb(transport)
  return adb
}
</script>

<template>
  <main>
    <mdui-dialog
      icon="report_gmailerrorred"
      headline="连接错误"
      :open="showErrorDialog"
      description="请检查手机和电脑之间的连接: "
      close-on-overlay-click
      >1. 手机是否已开启 ADB 调试 <br />
      2. 浏览器是否支持 WebUSB <br />
      3. 是否在浏览器中选择了正确的设备 <br />
      4. ADB 通信是否被其他进程占用 <br />
      <mdui-button slot="action" variant="text" @click="showErrorDialog = false">确定</mdui-button>
    </mdui-dialog>
    <mdui-dialog
      icon="broken_image"
      headline="连接断开"
      :open="showUnplugDialog"
      close-on-overlay-click
    >
      <mdui-button
        slot="action"
        variant="text"
        @click="(showUnplugDialog = false), (connected = false), (fabHitted = false)"
        >确定</mdui-button
      >
    </mdui-dialog>
    <mdui-fab
      id="fab"
      :loading="fabLoading"
      :disabled="connected"
      extended
      icon="broadcast_on_home"
      @click="
        connect()
          .then((result) => {
            adb = result.adb
            phoneInfo = reactive(result.phoneInfo)
            phoneStatus = reactive(result.phoneStatus)
            connected = result.connected
            screenCap = reactive(result.screenCap)
          })
          .catch((e) => {
            fabHitted = false
            showErrorDialog = true
          })
      "
      >{{ fabText }}</mdui-fab
    >

    <div id="card-container">
      <h3 id="place-holder" v-if="!connected">
        请将android手机连接到电脑, 确保“USB调试”开启, 点击“连接手机”
      </h3>
      <PhoneScrCapCard
        v-if="connected"
        :screen-cap="screenCap"
        @click="
          getScreenCap(adb).then((result) => {
            console.log(result)
            Object.assign(screenCap, result)
          })
        "
      />
      <PhoneInfoCard v-if="connected" :phone-info="phoneInfo" />
      <PhoneStatusCard v-if="connected" :phone-status="phoneStatus" />
    </div>
  </main>
</template>

<style>
#fab {
  position: fixed;
  bottom: 32px;
  right: 32px;
}
#card-container {
  display: flex;
  width: 100%;
  height: auto;
  gap: 25px;
  justify-content: center;
  align-items: center;
}
main {
  display: flex;
  width: 100%;
  height: auto;
  justify-content: center;
  align-items: center;
}
h3 {
  color: gray;
  left: 50%;
}
</style>
