<template>
  <!-- <div class="space-y-10 [&>*]:border-teal-500/20 [&>*]:border-l-2 [&>*]:pl-3"> -->
  <!-- <div class="space-y-10 [&>*]:shadow-md [&>*]:shadow-primary-800/10"> -->
  <div class="container space-y-10 ">

    <section>
      <h1 class="text-primary">AIDONS</h1>
      <h2 class="tracking-wider opacity-80 italic">Aide à l'instruction de documents par l'IA</h2>
    </section>

    <!-- toggle show history -->
    <div class="flex items-center gap-2">
      Historique
      {{ history }}
    </div>

    <p v-if="error" class="text-red-500">{{ error }}</p>

    <p class="text-sm text-gray-500 space-y-2">
      Essayer avec un exemple :
      <span v-for="code in barCodeExamples" :key="code" @click="decodedText = code"
        class="cursor-pointer text-primary-700 hover:text-primary-500 m-2">{{ code }}</span>

      <!-- Input principal - code barre -->
      <!-- <UInput v-model.lazy="decodedText" placeholder="Code barre" icon="i-lucide-barcode" size="xl" -->
      <UInput v-model="decodedText" placeholder="Code barre" icon="i-lucide-barcode" size="xl"
        class="[&>*]:tracking-[3px]" />
    </p>

    <!-- Recherche biblio.bnu et worldcat -->
    <p class="flex items-center gap-4 [&>*]:flex [&>*]:items-center [&>*]:gap-2">
      Voir sur
      <!-- biblio.bnu -->
      <a :href="links['biblio.bnu'] + decodedText" target="_blank"
        class="text-primary transition-all duration-500 underline" :aria-disabled="!decodedText"
        :class="{ 'opacity-30': !decodedText }">
        <UAvatar :src="getLogo(links['biblio.bnu'])" class="w-6 h-6" size="xs" :alt="'logo ' + links['biblio.bnu']" />
        Biblio.bnu
      </a>
      <!--  worldcat -->
      <a :href="`https://www.worldcat.org/search?q=${decodedText}`" target="_blank"
        class="text-primary transition-all duration-500 underline" :aria-disabled="!decodedText"
        :class="{ 'opacity-30': !decodedText }">
        <UAvatar :src="getLogo('https://www.worldcat.org')" class="w-6 h-6" size="xs"
          :alt="'logo ' + 'https://www.worldcat.org'" />
        WorldCat
      </a>
    </p>


    <!-- Notice ppn obtenue à partir de l'ISBN - api sudoc -->
    <div>
      <UDivider label="Notice" />
      <div v-if="sudocNotice" class="bg-blue-500/10 p-4 mt-1 rounded-lg">
        <ul>
          <li
            v-for="field in ppn && sudocNotice?.record?.datafield?.filter(field => ['200', /*'034'*/].includes(field['@_tag']))"
            :key="field">
            <p v-for="subfield in field.subfield" :key="subfield">
              <span class="text-primary"> ✓ </span> {{ subfield['#text'] }}
            </p>
          </li>
        </ul>
        <a v-if="ppn" :href="`https://www.sudoc.fr/${ppn}`" target="_blank" class="ml-1 text-primary underline">voir le
          détail sur le Sudoc</a>
      </div>
    </div>

    <div>
      <UDivider label="Bibliothèques où trouver le document" />
      <ul class="pl-3" :class="{ 'opacity-30': !decodedText }">
        <li v-for="lib in bibsData?.library" :key="lib" class="m-2"
          :class="{ 'bg-blue-500/10': lib.shortname === 'STRASBOURG-BNU' }">
          <!-- {{lib.rcr}} -->
          {{ lib.shortname }}
        </li>
      </ul>
    </div>

    <p>Placer un code barre devant la caméra Ou choisir un fichier dans la gallerie</p>
    <ImageBarcodeReader @decode="onDecode" @error="onError" class="text-primary-700" />
    <StreamBarcodeReader v-if="envProd" @decode="onDecode" @loaded="onLoaded" />

    <div>
      <span class="text-primary">Tous les liens</span>
      <ul class="pl-3 [&>*:hover]:text-primary transition-all" :class="{ 'opacity-30': !decodedText }">
        <li v-for="(value, key) in links" :key="key" class="flex items-center-center gap-2 m-2">
          <UAvatar :src="getLogo(value)" class="w-6 h-6" size="xs" :alt="'logo ' + value" />
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

  </div>


</template>

<script setup>
import { StreamBarcodeReader, ImageBarcodeReader } from "vue-barcode-reader";
import * as parser from 'fast-xml-parser';
import { useStorage } from '@vueuse/core';
const xmlParser = new parser.XMLParser({ ignoreAttributes: false });
const envProd = import.meta.env.PROD;


const error = ref('');
const decodedText = ref("");
// const decodedText = ref("9780545010221");

const links = {
  "biblio.bnu": 'https://biblio.bnu.fr/opac/query/',
  isbndb: 'https://isbndb.com/book/',
  google: 'https://www.google.com/search?q=',
  amazon: 'https://www.amazon.fr/s?k=',
  fnac: 'https://www.fnac.com/SearchResult/ResultList.aspx?SCat=0%211&Search=',
  livreRareBook: 'https://www.livre-rare-book.com/search/current.seam?isbn=',
  abebooks: 'https://www.abebooks.fr/servlet/SearchResults?isbn='
}

// API for cover image
const coverImageUrl = 'https://covers.openlibrary.org/b/isbn/';

// openLibrary API
const openLibUrl = 'https://openlibrary.org/api/books?bibkeys=ISBN:';
const url = computed(() => openLibUrl + decodedText.value + '&format=json&jscmd=data');
const { data: openLibData, error: openLiberror } = await useFetch(url);

// sudoc API - obtention de la notice bibliographique ppn à partir de l'ISBN
const header = { accept: 'application/json' };
const isbnToPpn = computed(() => `https://www.sudoc.fr/services/isbn2ppn/${decodedText.value}?format=json`);
const { data: sudocData, error: sudocError } = await useFetch(isbnToPpn, { headers: header, immediate: false});

const ppn = ref('');
watch(sudocData, data => {
  const jsonData = JSON.parse(data)
  ppn.value = jsonData?.sudoc?.query?.result?.ppn
})

const history = useStorage('history', [])

watch(ppn, newValue => {
  if (newValue && !history.value.includes(newValue)) 
    history.value.push(newValue)
})

// onMounted(() => {
//   fetch('https://aidons-backend.vercel.app/fapi')
//     .then(response => response.json())
//     .then(data => console.log(data));
// })

const sudocNoticeUrl = computed(() => ppn.value && `https://www.sudoc.fr/${ppn.value}.xml`);
const { data: sudocNotice } = useFetch(sudocNoticeUrl, { transform: (data) => xmlParser.parse(data), immediate: false });

const bibsUrl = computed(() => ppn.value && `https://www.sudoc.fr/services/multiwhere/${ppn.value}?format=json`)
const { data: bibsData, error: bibsError } = await useFetch(bibsUrl, { headers: header, transform: parseSudocResult, immediate: false });

function parseSudocResult(data) {
  try {
    return JSON.parse(data)?.sudoc.query.result
  } catch (error) {
    return null
  }
}

const barCodeExamples = [
  '9780545010221',
  // '9783161484100',
]

const getDomain = url => new URL(url).hostname
const getLogo = url => 'https://logo.clearbit.com/' + getDomain(url)

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


<style>
@keyframes appear {
  from {
    opacity: 0;
    scale: .8
  }

  to {
    opacity: 1;
    scale: 1
  }
}

.container>* {
  animation: appear linear both;
  animation-timeline: view();
  animation-range: entry 25% cover 50%;
}
@media (prefers-reduced-motion) {
  .container>* {
    animation: none;
  }
}

</style>