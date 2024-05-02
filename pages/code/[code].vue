<template>
  <main class="min-h-screen">
    <AppAccueil :code="code" />
  </main>
</template>

<script setup>

const route = useRoute()
const { code } = route.params


</script>