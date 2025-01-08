<template>

    <video ref="video" width="600" height="400" style="border: 1px solid black;"></video>

</template>


<script setup>


import { BrowserBarcodeReader } from '@zxing/library';


const video = ref(null);
const codeReader = new BrowserBarcodeReader();

const startCamera = async () => {   
  try {
    const constraints = {
      video: {
        facingMode: { ideal: "environment" } // Utiliser la caméra arrière si disponible
      }
    };

    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    video.value.srcObject = stream;

    codeReader.decodeFromVideoElement(video.value)
      .then(result => alert(result))
      .catch(err => console.error(err));
  } catch (error) {
    if (error.name === 'OverconstrainedError') {
      console.warn('Camera arrière non disponible, utilisation de la caméra par défaut.');
      // Essayer avec des contraintes plus flexibles
      const fallbackConstraints = { video: true };
      const fallbackStream = await navigator.mediaDevices.getUserMedia(fallbackConstraints);
      video.value.srcObject = fallbackStream;

      codeReader.decodeFromVideoElement(video.value)
        .then(result => alert(result))
        .catch(err => console.error(err));
    } else {
      console.error('Error accessing the camera:', error);
    }
  }
};

onMounted(() => {
  startCamera();
});
</script>