<template>
    <div>
        <qrcode-stream @detect="onDetect" :formats="ean_13"></qrcode-stream>
    </div>
    <p>Résultat : {{ result }}</p>
</template>


<script setup>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'
const onDetect = (decodedText) => {
    console.log(decodedText)
    result.value = decodedText
}
</script>