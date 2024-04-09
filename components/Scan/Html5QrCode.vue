<template>
    <div>
        <div id="qr-code-full-region">    
        </div>
        <p>Résultat : {{ result }}</p>
    </div>
</template>


<script setup>
import { Html5QrcodeScanner } from "html5-qrcode"
const result = ref('')
onMounted(() => {
    const html5QrCode = new Html5QrcodeScanner(
        "qr-code-full-region", { fps: 10, qrbox: 250 });
    html5QrCode.render(onScan);
})

const onScan = (decodedText) => {
    console.log(decodedText)
    result.value = decodedText
}
</script>