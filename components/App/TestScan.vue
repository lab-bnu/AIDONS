<template>
    <div>
        <qrcode-stream @detect="onDetect"></qrcode-stream>
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