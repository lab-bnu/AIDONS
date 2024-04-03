<template>
  <div class="space-y-5">

    <h1 class="text-primary">AIDONS</h1>
    <h2 class="tracking-wider opacity-80 italic">Aide à l'instruction de documents par l'IA</h2>

    <!--  input opening the camera on mobile -->
    <!-- <div class=" ">
        <input type="file" accept="image/*" capture="camera" @click="handleInput"
          class="bg-primary/50 text-white p-2 rounded-lg cursor-pointer" />
      </div> -->

    <div v-if="error">
      <p class="text-red-500">{{ error }}</p>
    </div>

    <p class="text-sm text-gray-500">
      Essayer avec un exemple : 
      <span v-for="code in barCodeExamples" :key="code" @click="decodedText = code" 
      class="cursor-pointer text-primary-700 hover:text-primary-500 m-2">{{ code }}</span>
    </p>
    <div class="flex gap-3 text-lg">
      <label for="code-barre">Code barre</label>
      <UInput v-model="decodedText" placeholder="Code barre" id="code-barre" icon = "i-lucide-barcode" />
    </div>

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
      <img :src="coverImage" alt="cover image" />
    </div>

    <div>
      <p class="text-primary">Résultat Open Library</p>
      <pre class="max-w-[100vw] overflow-scroll">{{ openLibData }}</pre>
    </div>


  </div>


</template>

<script setup>
import { StreamBarcodeReader, ImageBarcodeReader } from "vue-barcode-reader";
const error = ref('');
const decodedText = ref("");
// openLibrary API
const openLibUrl = 'https://openlibrary.org/api/books?bibkeys=ISBN:';
const url = computed(() => openLibUrl + decodedText.value + '&format=json&jscmd=data');
const { data: openLibData, error: openLiberror } = await useFetch(url);

// API for cover image
const coverImageUrl = 'https://covers.openlibrary.org/b/isbn/';
const coverImage = coverImageUrl + decodedText.value + '-M.jpg';

const barCodeExamples = [
  '9780545010221',
  // '9783161484100',
]

const links = {
  isbndb: 'https://isbndb.com/book/',
  sudoc: 'https://www.sudoc.fr/services/isbn2ppn/',
  google: 'https://www.google.com/search?q=',
  amazon: 'https://www.amazon.fr/s?k=',
  fnac: 'https://www.fnac.com/SearchResult/ResultList.aspx?SCat=0%211&Search=',
  livreRareBook: 'https://www.livre-rare-book.com/search/current.seam?isbn=',
  abebooks: 'https://www.abebooks.fr/servlet/SearchResults?isbn='
}

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
