<template>
    <main class="min-h-screen ">
        <section class=" mx-auto p-4 space-y-10">
            <h1 class="text-2xl font-bold">Historique de recherche</h1>
            <ul class="my-animate-children-appear" v-auto-animate >
                <!-- <ul class=""> -->
                <li v-for="item in history" :key="item.isbn"
                    class="flex items-center gap-3 mb-2 hover:!bg-slate-400/10 rounded-md odd:bg-gray-700/10 p-2 relative">
                    <span class="basis-50">
                        {{ item.insight }}
                    </span>
                    <a href="#" target="_blank" class="text-primary underline truncate md:w-80 w-72"> 
                        {{ item.isbn }}
                    </a>
                    <UButton @click="removeItem(history.indexOf(item))" icon="i-heroicons-trash" size="xs" color="primary" square variant="ghost" />
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