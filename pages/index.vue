<template>
  <main class="min-h-screen">
    <div class="space-y-24">

      <h1 class="text-primary">AIDONS</h1>
      <h2 class="tracking-wider opacity-80 italic">Aide au signalement par l'IA</h2>


      <!--  input opening the camera on mobile -->
      <div class="flex ">
       <input type="file" accept="image/*" capture="camera" @click="openCamera" class="bg-primary/50 text-white p-2 rounded-lg cursor-pointer" />
      </div>


    </div>
  </main>
</template>

<script setup>

// const 

const openCamera = () => {
  console.log('Opening camera')
  // Open the camera
  // This is a placeholder for the actual code that will open the camera
}

</script>
