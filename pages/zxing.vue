<template>


    <video id="video" width="600" height="400" style="border: 1px solid black;"></video>


</template>


<script setup>

onMounted(async () => {

    const script = document.createElement('script');
    script.src = 'https://unpkg.com/@zxing/library@latest';
    document.head.appendChild(script);

    await new Promise(r => setTimeout(r, 2000));

    async function startCamera() {
        try {
            const constraints = {
                video: {
                    facingMode: { exact: "environment" } // Forcer l'utilisation de la caméra arrière
                }
            };

            const stream = await navigator.mediaDevices.getUserMedia(constraints);
            const video = document.getElementById('video');
            video.srcObject = stream;

            const codeReader = new ZXing.BrowserBarcodeReader();
            codeReader.decodeFromVideoElement(video)
                .then(result => console.log(result))
                .catch(err => console.error(err));
        } catch (error) {
            console.error('Error accessing the camera:', error);
        }
    }

    startCamera();

});
</script>