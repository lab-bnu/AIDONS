<template>
    <main class=" ">
        <section class=" mx-auto p-4">
            <ul class="my-animate-children-appear" v-auto-animate >
                <!-- <UTable :rows="history" /> -->
                <li v-for="item in history.slice().reverse()" :key="item.isbn"
                    class="flex items-center gap-3 hover:!bg-slate-400/10 rounded-md odd:bg-gray-500/10 p-2 ">
                    <span class="basis-50">
                        {{ item.insight }}
                    </span>
                    <NuxtLink :to="`/code/${item.isbn}`" class="text-primary underline truncate w-40">
                        {{ item.isbn }}
                    </NuxtLink>
                    <UButton @click="removeItem(history.indexOf(item))" 
                        icon="i-heroicons-trash" size="xs" color="primary" variant="ghost" 
                        class="flex-1 justify-end" />
                </li>
            </ul>
            <div v-if = "!history.length">
                <p class="text-gray-500">Aucun historique de recherche</p>
            </div>
        </section>
    </main>
</template>

<script setup>

import { useStorage } from '@vueuse/core';
const history = useStorage('history', [])

const removeItem = (index) => {
    history.value.splice(index, 1)
}

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