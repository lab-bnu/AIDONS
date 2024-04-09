<template>
    <div>
        <!-- <qrcode-stream @detect="onDetect" :formats="ean_13"></qrcode-stream> -->
        <qrcode-stream @detect="onDetect" ></qrcode-stream>
        <p>Résultat : {{ result }}</p>
    </div>
</template>


<script setup>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'
const result = ref('')
const onDetect = (decodedText) => {
    console.log(decodedText)
    result.value = decodedText
}
</script>