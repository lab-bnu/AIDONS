<template>
    <div class="space-y-7">
        <qrcode-capture :formats @detect="onDetect">
        </qrcode-capture>
        <div class="flex items-center gap-2">
            <UToggle v-model="isCameraOpen" id = "toggle-camera" on-icon="i-lucide-camera" size = "xl"/>
            <label for = "toggle-camera">{{ isCameraOpen ? 'Désactiver' : 'Activer' }} la caméra</label>
        </div>
        <qrcode-stream v-if = "isCameraOpen" :formats @detect="onDetect">
        </qrcode-stream>
    </div>
</template>


<script setup>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'
const formats = ['ean_13', 'ean_8', 'upc_a', 'upc_e', 'code_39', 'code_128', 'code_93', 'itf', 'qr_code']

const props = defineProps({
    openCam: {
        type: Boolean,
        default: true
    }
})
const isCameraOpen = ref(props.openCam)

// define model decoded
const decoded = defineModel({
    type: String,
    default: ''
})

// const decoded = ref('')
const onDetect = (decodedObject) => {
    console.log('decodedObject', decodedObject)
    decoded.value = decodedObject[0]?.rawValue
}
</script>