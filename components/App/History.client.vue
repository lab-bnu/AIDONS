<template>
    <main class=" ">
        <div class="flex items-center px-4 my-3 gap-3">
            <UButton @click="favOnly = !favOnly" icon="i-lucide-book-marked" size="sm" color = "white" class="text-md !bg-primary-800/10" variant="ghost" 
            :class="{'[&>.i-lucide-book-marked]:text-primary-700' : favOnly}" label = "favoris"/>
            <UButton @click="download(csvConfig)(csv)" icon="i-lucide-download" size="sm" color = "white" class="text-md !bg-primary-800/10" variant="ghost" label = "csv"/>
        </div>
        <section class="mx-auto p-4">
            <ul class="my-animate-children-appear" v-auto-animate>
                <!-- <UTable :rows="history" /> -->
                <li v-for="item in (favOnly ? favHistory : history).slice().reverse()" :key="item.date"
                    class="flex items-center gap-4 rounded-md odd:bg-gray-500/10 p-2 mb-2" :class="{'!bg-primary/25' : item.bnu}">
                    <UIcon @click="toggleFav(history.indexOf(item))" :class="{' opacity-30': !item.fav}" name = "i-lucide-book-marked" class="basis- w-4 h-4" />
                    <span class="basis-52 break-all ">
                        {{ item.titre ?? item.insight }}
                    </span>
                    <NuxtLink :to="`/code/${item.isbn}`" class="text-primary underline truncate w-24 basis-10">
                        <!-- {{ item.isbn }} -->
                        Voir
                    </NuxtLink>
                    <UButton @click="removeItem(history.indexOf(item))" 
                        icon="i-heroicons-trash" size="xs" variant="link" color = "red"
                        class="flex-1 justify-end opacity-70" />
                </li>
            </ul>
            <div class="text-gray-500">
                <div v-if = "!favOnly && !history.length">
                    Aucun historique de recherche
                </div>
                <div v-else-if = "favOnly && !favHistory.length">
                    Aucun favori pour le moment
                </div>
            </div>
        </section>
    </main>
</template>

<script setup>

import { useStorage } from '@vueuse/core';
import { mkConfig, generateCsv, download } from "export-to-csv";
const csvConfig = mkConfig({ useKeysAsHeaders: true });
const csv = computed(() =>generateCsv(csvConfig)(history.value))

const history = useStorage('history', [])
const favOnly = ref(false)
const favHistory = computed(() => history.value.filter(item => item.fav))

const removeItem = index => history.value.splice(index, 1)

const toggleFav = index => history.value[index].fav = !history.value[index].fav

const handleFav = (elt, index) => {
    console.log('handleFav', elt, index)
    toggleFav(index)

}

const props = defineProps({
  max: {
    type: Number,
    default: 0
  }
})

</script>


<style scoped>
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