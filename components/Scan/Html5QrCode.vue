<template>
    <div id="qr-code-full-region"></div>
</template>


<script setup>
import { Html5QrcodeScanner } from "html5-qrcode"
const decoded = defineModel({
    type: String,
    default: ''
})

onMounted(() => {
    const html5QrCode = new Html5QrcodeScanner(
        "qr-code-full-region", { fps: 10, qrbox: 250, aspectRatio: 1.0, videoConstraints: { facingMode: "environment" } });
    html5QrCode.render(onScan);
})

const onScan = (decodedText) => {
    console.log(decodedText)
    decoded.value = decodedText
}
</script>