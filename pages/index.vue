<template>
  <main class="min-h-screen">
    <!-- <AppBarCode/> -->
    <AppAccueil/>
  </main>
</template>

<script setup>

const description =  "L'IA vous aide dans le processus de gestion des dons";

useSeoMeta({
  title: "AIDONS - Aide au signalement par l'IA",
  description,
  ogTitle: "AIDONS - Aide au signalement par l'IA",
  ogDescription: description,
  ogImage: "https://lab-aidons.vercel.app/app.png",
});

</script> 