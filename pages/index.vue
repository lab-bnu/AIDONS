<template>
  <main class="min-h-screen">
    <div class="space-y-5">

      <h1 class="text-primary">AIDONS</h1>
      <h2 class="tracking-wider opacity-80 italic">Aide au signalement par l'IA</h2>

      <!--  input opening the camera on mobile -->
      <!-- <div class=" ">
        <input type="file" accept="image/*" capture="camera" @click="handleInput"
          class="bg-primary/50 text-white p-2 rounded-lg cursor-pointer" />
      </div> -->

      <div v-if="error">
        <p class="text-red-500">{{ error }}</p>
      </div>

      <div class="flex gap-3 text-lg">
        <p class="">Code barre décodé</p>
        <p class="text-primary">{{ decodedText }}</p>
      </div>

      <p>Placer un code barre devant la caméra Ou choisir un fichier dans la gallerie</p>
      <ImageBarcodeReader @decode="onDecode" @error="onError" class="text-primary-700"></ImageBarcodeReader>
      <StreamBarcodeReader @decode="onDecode" @loaded="onLoaded"></StreamBarcodeReader>
    </div>
  </main>
</template>

<script setup>
import { StreamBarcodeReader, ImageBarcodeReader } from "vue-barcode-reader";
const decodedText = ref("______");
const error = ref('');

const onDecode = (decodedValue) => {
  decodedText.value = decodedValue
}

const onError = (error) => {
  error.value = error
}

const onLoaded = () => {
  console.log('Barcode reader loaded')
}

const handleInput = () => {
  console.log('handleInput')
}

</script>
