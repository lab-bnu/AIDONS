<template>
  <div class="container space-y-10 ">

    <section>
      <h1 class="text-primary">AIDONS</h1>
      <h2 class="tracking-wider opacity-80 italic">Aide à l'instruction de documents par l'IA</h2>
    </section>

    <p v-if="error" class="text-red-500">{{ error }}</p>

    <p class="text-sm text-gray-500 space-y-2">
      Exemples :
      <span v-for="code in barCodeExamples" :key="code" @click="decodedText = code"
        class="cursor-pointer text-primary-700 hover:text-primary-500 m-2">{{ code }}</span>

      <!-- Input principal - code barre -->
      <UInput v-model="decodedText" placeholder="Code barre" icon="i-lucide-barcode" size="xl"
        class="[&>*]:tracking-[3px]" />
    </p>

    <p class="flex items-center gap-4 [&>*]:flex [&>*]:items-center [&>*]:gap-2">
      Voir sur
      <!-- Biblio.bnu -->
      <a :href="links['Biblio.bnu'] + decodedText" target="_blank"
        class="text-primary transition-all duration-500 underline" :aria-disabled="!decodedText"
        :class="{ 'opacity-30': !decodedText }">
        <UAvatar :src="getLogo(links['Biblio.bnu'])" class="w-6 h-6" size="xs" :alt="'logo ' + links['Biblio.bnu']" />
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
          <li v-for="field in tag200" :key="field">
            <p v-for="subfield in field.subfield" :key="subfield" ref="tag200">
              <span class="text-primary"> ✓ </span> {{ subfield['#text'] }}
            </p>
          </li>
        </ul>
        <a v-if="ppn" :href="`https://www.sudoc.fr/${ppn}`" target="_blank" class="ml-1 text-primary underline">
          voir le détail sur le Sudoc
        </a>
      </div>
    </div>

    <div>
      <UDivider label="BU où trouver le document" />
      <p v-show="decodedText" :class="{ 'text-primary': foundInBNU }">
        {{ foundInBNU ? " ✓ Document trouvé à la BNU" : "Document non trouvé à la BNU" }}
      </p>
      <UAccordion :variant="'ghost'"
        :items="[{ label: 'Afficher / masquer la liste', content: bibsData?.map(lib => lib.shortname).join(' | ') }]" />
    </div>

    <div>
      <UDivider label="Reconnaissance avec le backend" />
      <form @submit.prevent ="handleSubmit" class="flex items-center gap-2" ref = "backendForm"
        action="https://aidons-backend.vercel.app/barcode" enctype="multipart/form-data" method="post">
        <!--  inpur opens camera on mobile -->
        <UInput name="file" type="file" accept="image/*" capture="environment" :loading="waitingBackend" @change="handleSubmit"
          class="p-1 m-1 bg-gray-300/25 rounded-md" />
          {{ backendDtata }}
        <!-- <input type="submit"> -->
      </form>
    </div>

    <div>
      <UDivider label="Reconnaissance avec la caméra" />
      <AppTestScan v-model="decodedText" />
    </div>
    <!-- <ImageBarcodeReader @decode="onDecode" @error="onError" class="text-primary-700" />
    <StreamBarcodeReader v-if="envProd" @decode="onDecode" @loaded="onLoaded" /> -->

    <div>
      <span class="text-primary">Autres liens</span>
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
const toastNotif = useToast();
const xmlParser = new parser.XMLParser({ ignoreAttributes: false });
const envProd = import.meta.env.PROD;

const error = ref('');
const decodedText = ref("");

const barCodeExamples = [
  '9780545010221',
  '9780838911297'
  // '9783161484100',
]

const links = {
  "Biblio.bnu": 'https://biblio.bnu.fr/opac/query/',
  Isbndb: 'https://isbndb.com/book/',
  Google: 'https://www.google.com/search?q=',
  "Librairie Kleber": 'https://www.librairie-kleber.com/listeliv.php?base=paper&mots_recherche=',
  Amazon: 'https://www.amazon.fr/s?k=',
  Fnac: 'https://www.fnac.com/SearchResult/ResultList.aspx?SCat=0%211&Search=',
  LivreRareBook: 'https://www.livre-rare-book.com/search/current.seam?isbn=',
  Abebooks: 'https://www.abebooks.fr/servlet/SearchResults?isbn='
}

// API for cover image
const coverImageUrl = 'https://covers.openlibrary.org/b/isbn/';

// openLibrary API
const openLibUrl = 'https://openlibrary.org/api/books?bibkeys=ISBN:';
const url = computed(() => openLibUrl + decodedText.value + '&format=json&jscmd=data');
const { data: openLibData, error: openLiberror } = await useFetch(url);

// sudoc API - obtention de la notice Bibliographique ppn à partir de l'ISBN
const header = { accept: 'application/json' };
const parsePPN = data => JSON.parse(data)?.sudoc?.query?.result?.ppn ?? null;
const isbnToPpn = computed(() => `https://www.sudoc.fr/services/isbn2ppn/${decodedText.value}?format=json`);
const { data: ppn, error: sudocError } = await useFetch(isbnToPpn, { headers: header, immediate: false, transform: parsePPN });


// onMounted(() => {

//   fetch('https://aidons-backend.vercel.app/extractinfo')
//     .then(response => response.json())
//     .then(data => console.log(data));
// })

const sudocNoticeUrl = computed(() => ppn.value && `https://www.sudoc.fr/${ppn.value}.xml`);
const { data: sudocNotice } = useFetch(sudocNoticeUrl, { transform: (data) => xmlParser.parse(data), immediate: false });
const tag200 = computed(() => sudocNotice.value?.record?.datafield?.filter(field => field['@_tag'] === '200'))

const bibsUrl = computed(() => ppn.value && `https://www.sudoc.fr/services/multiwhere/${ppn.value}?format=json`)
const { data: bibsData, error: bibsError } = await useFetch(bibsUrl, { headers: header, immediate: false, transform: parseSudocResult });

function parseSudocResult(data) {
  try {
    const libraries = JSON.parse(data)?.sudoc.query.result.library
    return Array.isArray(libraries) ? libraries : [libraries]
  } catch (error) {
    return null
  }
}

// Notif si document trouvé à la BNU
const foundInBNU = computed(() => bibsData.value?.some(lib => lib.shortname === 'STRASBOURG-BNU'))
watch(foundInBNU, newValue => {
  if (newValue) toastNotif.add({ title: 'Document trouvé à la BNU', type: 'success' })
})

// historique de recherche - à chque notice trouvée
const history = useStorage('history', [])
watch(sudocNotice, newValue => {
  if (newValue)
    toastNotif.add({ title: 'Notice sudoc trouvée', type: 'success' })
  if (newValue && !history.value.includes(decodedText.value))
    history.value.push({
      isbn: decodedText.value,
      ppn: ppn.value,
      date: new Date().toLocaleString(),
      insight: tag200.value[0].subfield[0]['#text'] ?? null   // insight : titre de la notice (tag 200)
    })
})

// Affiche une notif lorsqu'un nouveau code barre est détecté
const lastNotif = ref('')
watchEffect(() => {
  if (decodedText.value && decodedText.value !== lastNotif.value) {
    lastNotif.value = decodedText.value
    toastNotif.add({ title: `Code barre détecté - ${decodedText.value}`, type: 'success' })
  }
})

const getDomain = url => new URL(url).hostname
const getLogo = url => 'https://logo.clearbit.com/' + getDomain(url)

const onDecode = decodedValue => decodedText.value = decodedValue
const onError = err => error.value = err

const onLoaded = () => {
  console.log('Barcode reader loaded')
}

const backendForm = ref(null)
const waitingBackend = ref(false)
const backendDtata = ref(null)
const handleSubmit = () => {
  waitingBackend.value = true
  console.log('handleSubmit', backendForm.value)
  // send file to baxkend
  fetch('https://aidons-backend.vercel.app/barcode', {
    method: 'POST',
    body: new FormData(backendForm.value),
  })
    .then(response => response.json())
    .then(data => {
      backendDtata.value = data
      data && data.code && (decodedText.value = data.code)
    })
    .catch(err => error.value = err)
    .finally(() => waitingBackend.value = false)
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