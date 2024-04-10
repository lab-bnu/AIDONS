<template>
    <div>
        <qrcode-stream @detect="onDetect"
            :formats="['ean_13', 'ean_8', 'upc_a', 'upc_e', 'code_39', 'code_128', 'code_93', 'itf', 'qr_code']">
        </qrcode-stream>
        <br>
        <qrcode-capture
            :formats="['ean_13', 'ean_8', 'upc_a', 'upc_e', 'code_39', 'code_128', 'code_93', 'itf', 'qr_code']"
            @detect="onDetect">
        </qrcode-capture>
        <!-- <p>Résultat : {{ decoded }}</p> -->
    </div>
</template>


<script setup>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'

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