<template>
    <div class="space-y-7">
        <qrcode-capture :formats @detect="onDetect">
        </qrcode-capture>
        <qrcode-stream :formats @detect="onDetect">
        </qrcode-stream>
    </div>
</template>


<script setup>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'
const formats = ['ean_13', 'ean_8', 'upc_a', 'upc_e', 'code_39', 'code_128', 'code_93', 'itf', 'qr_code']

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