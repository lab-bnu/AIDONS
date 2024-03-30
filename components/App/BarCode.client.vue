<template>
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

    <div>
      <p class="text-primary"> Image de couverture</p>
      <img :src="coverImage" alt="cover image" />
    </div>

    <div>
      <p class="text-primary">Résultat Open Library</p>
      <pre>{{ openLibData }}</pre>
    </div>

<div>
Liens
<a :href ='"
https://isbndb.com/book/"+decodedText' >isbndb</a>
<a :href ='"https://www.sudoc.fr/services/isbn2ppn/"+decodedText' >sudoc - ppn</a>
</div>

  </div>


</template>

<script setup>
import { StreamBarcodeReader, ImageBarcodeReader } from "vue-barcode-reader";
// const decodedText = ref("9780545010221");
const decodedText = ref("");
// const decodedText = ref("9783161484100");
const error = ref('');

// openLibrary API
const openLibUrl = 'https://openlibrary.org/api/books?bibkeys=ISBN:';
const url = computed(() => openLibUrl + decodedText.value + '&format=json&jscmd=data');
const { data: openLibData, error: openLiberror } = await useFetch(url);

// API for cover image
const coverImageUrl = 'https://covers.openlibrary.org/b/isbn/';
const coverImage = coverImageUrl + decodedText.value + '-M.jpg';

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
