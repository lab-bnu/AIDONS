<template>
    <div>
        <qrcode-stream 
            @detect="onDetect"  
            :formats="['ean_13', 'ean_8', 'upc_a', 'upc_e', 'code_39', 'code_128', 'code_93', 'itf', 'qr_code' ]">
        </qrcode-stream>
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