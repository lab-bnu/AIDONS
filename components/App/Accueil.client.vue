<template>
  <div class="space-y-7 ">

    <section>
      <h1 class="text-primary text-2xl flex items-center gap-2">
        <UIcon name="i-lucide-library" class="mb-[2px]"/>AIDONS</h1>
      <h2 class="opacity-80 mt-1 text-xl">Aide à l'instruction de documents</h2>
    </section>

    <p v-if="error" class="text-red-500">{{ error }}</p>

    <p v-if="!envProd" class="text-sm text-gray-500 space-y-2">
      Exemples (mode développement ) :
      <span v-for="code in barCodeExamples" :key="code" @click="decodedText = code"
        class="cursor-pointer text-primary-700 hover:text-primary-500 m-2">{{ code }}
      </span>

    </p>
    <!-- Input principal - code barre -->
    <div class="relative">

      <UInput v-model="decodedText" placeholder="Saisir un isbn ou scanner" icon="i-lucide-barcode" size="xl" :variant = "'none'" 
        :ui = "{variant: { none: 'bg-blue-950/70 !text-white'} }"/>

      <a :href="links['Biblio.bnu'] + decodedText" target="_blank"
        alt = "voir sur Biblio.bnu" title = "voir sur Biblio.bnu"
        class="absolute top-[6px] right-3" :aria-disabled="!decodedText"
        :class="{ 'opacity-30': !decodedText }">
        <UAvatar :src="getLogo(links['Biblio.bnu'])"  size="sm" :alt="'logo ' + links['Biblio.bnu']" imgClass = "border-2 border-gray-200 p-[2px]" />
      </a>
    </div>

    <AppTestScan v-model="decodedText" :open-cam="!props.code" /> <!-- cam active par défaut sauf code en paramètre de l'url -->

    <!-- Notice ppn obtenue à partir de l'ISBN - api sudoc -->
    <div>
      <UDivider label="Notice" />
      <!-- <div v-if="sudocNotice" class="bg-blue-500/10 p-4 mt-1 rounded-lg"> -->
      <div v-if="sudocNotice" class="bg-gradient-to-tr from-gray-300/20 to-transparent p-4 mt-1 rounded-lg">
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
      <!-- <UDivider label="BU où trouver le document" /> -->
      <p v-show="decodedText" :class="{ 'text-primary': foundInBNU }">
        {{ foundInBNU ? " ✓ Document trouvé à la BNU" : "Document non trouvé à la BNU" }}
      </p>
      <UAccordion :variant="'ghost'"
        :items="[{ label: 'Afficher / masquer la liste', content: bibsData?.map(lib => lib.shortname).join(' | ') }]" />
    </div>
    <div>
      <UDivider label="Historique" />
      <AppHistory />
    </div>


    <!-- <p class="flex items-center gap-4 [&>*]:flex [&>*]:items-center [&>*]:gap-2"> -->
    <p class="hidden">
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

    <div class="hidden">
      <UDivider label="Reconnaissance avec le backend" />
      <form @submit.prevent="handleSubmit" class="flex items-center gap-2" ref="backendForm"
        action="https://aidons-backend.vercel.app/barcode" enctype="multipart/form-data" method="post">
        <UInput name="file" type="file" accept="image/*" capture="environment" :loading="waitingBackend"
          @change="handleSubmit" class="p-1 m-1 bg-gray-300/25 rounded-md" />
      </form>
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

const props = defineProps({
  code: {
    type: String,
    default: ''
  }
})

const decodedText = ref(props.code);
const error = ref('');

const barCodeExamples = [
  '9780545010221',
  '9780838911297'
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

// API pour récupérer l'image de couverture
const coverImageUrl = 'https://covers.openlibrary.org/b/isbn/';

// API OpenLibrary 
const openLibUrl = 'https://openlibrary.org/api/books?bibkeys=ISBN:';
const url = computed(() => openLibUrl + decodedText.value + '&format=json&jscmd=data');
const { data: openLibData, error: openLiberror } = await useFetch(url);

// API SUDOC 
// ppn à partir de l'ISBN
const header = { accept: 'application/json' };
const parsePPN = data => JSON.parse(data)?.sudoc?.query?.result?.ppn ?? null;
const isbnToPpn = computed(() => `https://www.sudoc.fr/services/isbn2ppn/${decodedText.value}?format=json`);
const { data: ppn, error: sudocError, execute: fetchPpn } = await useFetch(isbnToPpn, { headers: header, immediate: false, transform: parsePPN });
// lance la requête dès chargement si code barre passé en paramètre
if (props.code) {
  fetchPpn()
}

// notice bibliographique à partir du ppn
const sudocNoticeUrl = computed(() => ppn.value && `https://www.sudoc.fr/${ppn.value}.xml`);
const { data: sudocNotice } = useFetch(sudocNoticeUrl, { transform: (data) => xmlParser.parse(data), immediate: false });
const tag200 = computed(() => sudocNotice.value?.record?.datafield?.filter(field => field['@_tag'] === '200'))

// BU où trouver le document
const bibsUrl = computed(() => ppn.value && `https://www.sudoc.fr/services/multiwhere/${ppn.value}?format=json`)
const { data: bibsData, pending: bibsPending } = await useFetch(bibsUrl, { headers: header, immediate: false, transform: parseSudocResult });

function parseSudocResult(data) {
  try {
    const libraries = JSON.parse(data)?.sudoc.query.result.library
    return Array.isArray(libraries) ? libraries : [libraries]
  } catch (error) {
    return null
  }
}

// Notif si document trouvé à la BNU
const checkFoundBnu = () => !bibsPending.value && bibsData.value?.some(lib => lib.shortname === 'STRASBOURG-BNU')
const foundInBNU = computed(checkFoundBnu);
watch(foundInBNU, newValue => {
  if (newValue) toastNotif.add({ title: 'Document trouvé à la BNU', type: 'success', icon: 'i-lucide-book-open-check' })
})

const bookTitle = computed(() => tag200.value[0].subfield[0]['#text'] ?? null)  // titre de la notice (tag 200)
// historique de recherche - à chque notice trouvée
const history = useStorage('history', [])
watch(bibsPending, async newValue => {
  console.log('watch sudocNotice', newValue, 'found in BNU', foundInBNU.value)
  if (! newValue && sudocNotice.value) {
    await nextTick()
    toastNotif.add({ title: `Notice trouvée - ${bookTitle.value}`, type: 'success', icon: 'i-lucide-book' })
    history.value.push({
      isbn: decodedText.value,
      ppn: ppn.value,
      date: new Date().toLocaleString(),
      bnu: foundInBNU.value,
      insight: bookTitle.value
    })
  }
})

// Affiche une notif lorsqu'un nouveau code barre est détecté
const lastNotif = ref('')
watchEffect(() => {
  if (decodedText.value && decodedText.value !== lastNotif.value) {
    lastNotif.value = decodedText.value
    // Notif code détecté sauf saisie manuelle
    const focused = document.activeElement;
    if (focused?.type !== 'text') 
      toastNotif.add({ title: `Code barre détecté - ${decodedText.value}`, type: 'success' })
  }
})

const getDomain = url => new URL(url).hostname
const getLogo = url => 'https://logo.clearbit.com/' + getDomain(url)

// Backend 
const backendForm = ref(null)
const waitingBackend = ref(false)
const handleSubmit = () => {
  waitingBackend.value = true
  fetch('https://aidons-backend.vercel.app/barcode', {
    method: 'POST',
    body: new FormData(backendForm.value),
  })
    .then(response => response.json())
    .then(data => data?.code && (decodedText.value = data.code))
    .catch(err => error.value = err)
    .finally(() => waitingBackend.value = false)
}

</script>
