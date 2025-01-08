<template>


    <video id="video" width="600" height="400" style="border: 1px solid black;"></video>


</template>


<script setup>

onMounted(async () => {

const script = document.createElement('script');
script.src = 'https://unpkg.com/@zxing/library@latest';
document.head.appendChild(script);

await new Promise(r => setTimeout(r, 2000));

const codeReader = new ZXing.BrowserBarcodeReader();
codeReader.getVideoInputDevices()
    .then(videoInputDevices => {
        codeReader.decodeFromInputVideoDevice(videoInputDevices[0] ? videoInputDevices[0].deviceId : undefined, 'video')
            .then(result => alert(result))
            .catch(err => console.error(err));
    })
    .catch(err => console.error(err));

})
</script>