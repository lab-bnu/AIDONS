<template>
  <div class="space-y-5">

    <h1 class="text-primary">AIDONS</h1>
    <h2 class="tracking-wider opacity-80 italic">Aide à l'instruction de documents par l'IA</h2>

    <!--  input opening the camera on mobile -->
    <!-- <div class=" ">
        <input type="file" accept="image/*" capture="camera" @click="handleInput"
          class="bg-primary/50 text-white p-2 rounded-lg cursor-pointer" />
      </div> -->

    <p v-if="error" class="text-red-500">{{ error }}</p>

    <p class="text-sm text-gray-500">
      Essayer avec un exemple : 
      <span v-for="code in barCodeExamples" :key="code" @click="decodedText = code" 
      class="cursor-pointer text-primary-700 hover:text-primary-500 m-2">{{ code }}</span>
    </p>

    <UInput v-model="decodedText" placeholder="Code barre" icon = "i-lucide-barcode" size = "xl" class="[&>*]:tracking-widest" />

    <p>Placer un code barre devant la caméra Ou choisir un fichier dans la gallerie</p>
    <ImageBarcodeReader @decode="onDecode" @error="onError" class="text-primary-700"></ImageBarcodeReader>
    <StreamBarcodeReader @decode="onDecode" @loaded="onLoaded"></StreamBarcodeReader>

    <div>
      <span class="text-primary">Liens</span>
      <ul class="pl-3 [&>*:hover]:text-primary transition-all" :class="{'opacity-50' : !decodedText}">
        <li v-for="(value, key) in links" :key="key" class="flex items-center-center gap-2 m-2">
          <UAvatar :src="'https://logo.clearbit.com/' + getDomain(value)" class="w-6 h-6" size = "xs" :alt = "'logo ' + value"/>
          <a :href="value + decodedText" target="_blank">{{ key }}</a>
        </li>
      </ul>
    </div>

    <div>
      <p class="text-primary"> Image de couverture</p>
      <img :src="coverImageUrl + decodedText + '-M.jpg'" alt="cover" />
    </div>

    <div v-if="openLibData">
      <p class="text-primary">Résultat Open Library</p>
      <pre class="max-w-[100vw] overflow-x-scroll">{{ openLibData }}</pre>
    </div>
<!-- 
    <div v-if="isbndbData?.length">
      <p class="text-primary">Résultat ISBN DB</p>
      <pre class="max-w-[100vw] overflow-scroll">{{ isbndbData }}</pre>
    </div>

    <div v-if="isbnDbError">
      <p class="text-red-500">{{ isbnDbError }}</p>
    </div> -->

  </div>


</template>

<script setup>
import { StreamBarcodeReader, ImageBarcodeReader } from "vue-barcode-reader";
const error = ref('');
const decodedText = ref("");
// const decodedText = ref("9780545010221");

const links = {
  isbndb: 'https://isbndb.com/book/',
  sudoc: 'https://www.sudoc.fr/services/isbn2ppn/',
  google: 'https://www.google.com/search?q=',
  amazon: 'https://www.amazon.fr/s?k=',
  fnac: 'https://www.fnac.com/SearchResult/ResultList.aspx?SCat=0%211&Search=',
  livreRareBook: 'https://www.livre-rare-book.com/search/current.seam?isbn=',
  abebooks: 'https://www.abebooks.fr/servlet/SearchResults?isbn='
}

// openLibrary API
const openLibUrl = 'https://openlibrary.org/api/books?bibkeys=ISBN:';
const url = computed(() => openLibUrl + decodedText.value + '&format=json&jscmd=data');
const { data: openLibData, error: openLiberror } = await useFetch(url);
const {data : isbndbData, error: isbnDbError} = await useFetch(computed(() =>links.sudoc + decodedText.value));

// API for cover image
const coverImageUrl = 'https://covers.openlibrary.org/b/isbn/';

const barCodeExamples = [
  '9780545010221',
  // '9783161484100',
]

const getDomain = (url) => new URL(url).hostname

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
