<template>
    <div class="space-y-7">
    <div class="bg-slate-500/25 p-2 rounded-lg max-h-[60vh] overflow-hidden">
        <qrcode-stream v-if="isCameraOpen" :formats @detect="onDetect" />
    </div>
        <!-- <div class="flex items-center gap-2">
            <UToggle v-model="isCameraOpen" id = "toggle-camera" on-icon="i-lucide-camera" size = "xl"/>
            <label for = "toggle-camera" class="cursor-pointer">{{ isCameraOpen ? 'Désactiver' : 'Activer' }} la caméra</label>
        </div> -->
        <div class="flex justify-between">
            <!-- :icon="'i-lucide-cloud-upload'" -->
            <UButton @click="triggerFileInput" title="Importer un fichier" alt="Importer un fichier"
                class="bg-slate-500/25 w-24 h-24 justify-center" size="xl" rounded :icon="'i-lucide-upload'"
                color="primary" variant="ghost"></UButton>

            <UButton @click="isCameraOpen = !isCameraOpen" class="bg-slate-500/25 w-24 h-24 justify-center"
                title="Activer / déscativer la caméra" alt="Activer ou désactiver la caméra" size="xl" rounded
                :icon="isCameraOpen ? 'i-lucide-camera-off' : 'i-lucide-camera'" color="primary" variant="ghost">
            </UButton>

            <NuxtLink to="/historique" class="bg-slate-500/25 w-24 h-24 justify-center rounded-md" >
                <UButton title="Historique" alt="Historique" class="w-24 h-24 justify-center" size="xl" rounded
                    :icon="'i-lucide-history'" color="primary" variant="ghost" />
            </NuxtLink>
        </div>
        <qrcode-capture :formats @detect="onDetect" ref="fileInput" class="hidden"></qrcode-capture>
    </div>
</template>


<script setup>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'
const fileInput = ref(null)
const formats = ['ean_13', 'ean_8', 'upc_a', 'upc_e', 'code_39', 'code_128', 'code_93', 'itf', 'qr_code']

const props = defineProps({
    openCam: {
        type: Boolean,
        default: true
    }
})
const isCameraOpen = ref(props.openCam)

const decoded = defineModel({
    type: String,
    default: ''
})

function triggerFileInput() {
    console.log('triggerFileInput', fileInput.value)
    // log root element of the component
    console.log('root element', fileInput.value.$el)
    fileInput.value?.$el.click()
}

const onDetect = (decodedObject) => {
    console.log('decodedObject', decodedObject)
    decoded.value = decodedObject[0]?.rawValue
}
</script>