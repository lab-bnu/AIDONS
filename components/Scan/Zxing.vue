<template>
    <video ref="video" width="600" height="400" style="border: 1px solid black;"></video>
</template>


<script setup>

import { BrowserBarcodeReader, BarcodeFormat } from '@zxing/library';

const decoded = defineModel({
    type: String,
    default: ''
})

// mettre en place les hints
const hints = new Map();
hints.set(BarcodeFormat.EAN_13, {})

const video = ref(null);
const codeReader = new BrowserBarcodeReader()

const startCamera = async () => {
    const constraints = {
        video: {
            facingMode: { ideal: "environment" } // Utiliser la caméra arrière si disponible
        }
    };

    let stream = await navigator.mediaDevices.getUserMedia(constraints);
    video.value.srcObject = stream;

    codeReader.decodeFromVideoElement(video.value)
        .then(result => { decoded.value = result.text; 
            codeReader.reset(); 
            stream.getTracks().forEach(track => track.stop());
            startCamera();
            

        })
        .catch(err => console.error(err));

};



onMounted(() => {
    startCamera();
});

</script>