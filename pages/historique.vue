<template>
    <main class="min-h-screen">
        <h1 class="text-2xl font-bold text-center">Historique</h1>

        <AppHistory />
    </main>
</template>

<script setup>

const description = "Affichage et gestion de l'historique de recherche";

useSeoMeta({
    title: "Mon historique - AIDONS",
    description,
    ogTitle: "Mon historique - AIDONS",
    ogDescription: description,
    ogImage: "https://lab-aidons.vercel.app/app.png",
});

</script>