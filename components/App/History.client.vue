<template>
    <main class=" ">
        <div class="flex items-center px-4 my-5 gap-2">
            <UButton label = "favoris"  @click="favOnly = !favOnly" 
                icon="i-lucide-book-marked" size="sm" :color = "favOnly ? 'primary' : 'white'" class="text-md !bg-primary-800/10" variant="ghost"/>
            <UButton label = "csv"      @click="download(csvConfig)(csv)" 
                icon="i-lucide-download" size="sm" color = "white" class="text-md !bg-primary-800/10" variant="ghost"/>
            <UInput :placeholder = "`Rechercher ${favOnly ? 'parmi les favoris' : ''}`" icon="i-lucide-search" size="sm" v-model="search" class="flex-1"/>
        </div>
        <section class="mx-auto p-4">
            <ul class="my-animate-children-appear" v-auto-animate>
                <li v-for="item in filteredShownHistory.slice().reverse()" :key="item.date"
                    class="flex items-center gap-4 rounded-md odd:bg-gray-500/10 p-2 mb-2" :class="{'!bg-primary/25' : item.bnu}">
                    <UIcon @click="item.fav = !item.fav" name = "i-lucide-book-marked" 
                        :class="{'my-animation-pulse': item.fav, 'opacity-30': !item.fav}" />
                    <span class="basis-52 break-all flex-1">
                        {{ item.titre ?? item.insight }}
                    </span>
                    <NuxtLink :to="`/code/${item.isbn}`" class="text-primary underline truncate w-24 basis-10">
                        Voir
                    </NuxtLink>
                    <UButton @click="removeItem(history.indexOf(item))" 
                        icon="i-heroicons-trash" size="xs" variant="link" color = "red"
                        class="ml-auto justify-self-end justify-end opacity-70" />
                </li>
            </ul>
            <div v-if = "!filteredShownHistory.length" class="text-gray-500">
                {{ favOnly ? 'Aucun favori trouvé' : 'Aucun historique trouvé'}}
            </div>
        </section>

        <section>
            <!-- <iframe class="m-auto"src = "https://bnufr-my.sharepoint.com/:o:/g/personal/arthurb_bnu_fr/EgPXTIlLlKhJgSW6QzdhB-4BY54aP5v-Kg7-aQGEsj68fQ?e=tFF1zo"></iframe>
            <iframe class="m-auto"src = "https://bnufr-my.sharepoint.com/:o:/g/personal/arthurb_bnu_fr/EgPXTIlLlKhJgSW6QzdhB-4BY54aP5v-Kg7-aQGEsj68fQ?e=T6ezCE">
            </iframe> -->
        </section>
    </main>
</template>

<script setup>

import { mkConfig, generateCsv, download } from "export-to-csv";
const csvConfig = mkConfig({ useKeysAsHeaders: true, fieldSeparator: ';', filename: 'AIdons', useBom: true });
const csv = computed(() =>generateCsv(csvConfig)(filteredShownHistory.value))

import { useStorage } from '@vueuse/core';
const history = useStorage('history', [])
const favOnly = ref(false)
const filterSearch = item => item.titre?.toLowerCase().includes(search.value.toLowerCase()) 
const favHistory = computed(() => history.value.filter(item => item.fav))
const shownHistory = computed(() => favOnly.value ? favHistory.value : history.value)
const filteredShownHistory = computed(() => !search.value ? shownHistory.value : shownHistory.value.filter(filterSearch))

const removeItem = index => history.value.splice(index, 1)

const search = ref('')

const props = defineProps({
  max: {
    type: Number,
    default: 0
  }
})

</script>


<style scoped>


.my-animation-pulse {
    animation: pulse .3s ease;
  }
  
  @keyframes pulse {
    0% {
        transform: scale(1);
    }
    50% {
        transform: scale(1.6);
    }
    100% {
        transform: scale(1);
    }
}


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

.my-animate-children-appear>li {
    animation: appear linear both;
    animation-timeline: view();
    animation-range: entry 50% cover 25%;
}

@media (prefers-reduced-motion) {
    .my-animate-children-appear>* {
        animation: none;
    }
}
</style>