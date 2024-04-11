<template>
  <main class="min-h-screen">
    <AppBarCode :code="code" />
  </main>
</template>

<script setup>

const route = useRoute()
const { code } = route.params


</script>