<template>
    <div class="space-y-7">
        <div class=" relative p- rounded-lg w-full h-[40vh] overflow-hidden grid place-items-center bg-gradient-to-tr from-blue-950/20 to-blue-950/40">
            <div class = "absolute w-full h-full z-30 border-[10px] border-blue-300/20"></div>
            <qrcode-stream v-if="isCameraOpen" :formats @detect="onDetect" class="!absolute z-20 w-full h-full"/>
            <UIcon v-else name="i-lucide-book-open" class="w-24 h-24 text-gray-300"/>
        </div>
        <!-- <div class="flex items-center gap-2">
            <UToggle v-model="isCameraOpen" id = "toggle-camera" on-icon="i-lucide-camera" size = "xl"/>
            <label for = "toggle-camera" class="cursor-pointer">{{ isCameraOpen ? 'Désactiver' : 'Activer' }} la caméra</label>
        </div> -->
        <div class="flex justify-between">
            <NuxtLink to="/historique" class="w-24 h-24 justify-center rounded-md ">
                <UButton title="Historique" alt="Historique" class="w-24 h-24 justify-center bg-gradient-to-tr from-blue-950 to-blue-950/70" rounded variant="ghost">
                    <UIcon name="i-lucide-history" class="w-10 h-10" />
                </UButton>
            </NuxtLink>
            <!-- :icon="'i-lucide-cloud-upload'" -->
            <UButton @click="triggerFileInput" title="Importer un fichier" alt="Importer un fichier"
                class=" w-24 h-24 justify-center bg-gradient-to-tr from-blue-950 to-blue-950/70" rounded variant="ghost">
                <UIcon name="i-lucide-upload" class="w-10 h-10" />
            </UButton>
            <UButton @click="isCameraOpen = !isCameraOpen" title="Activer / déscativer la caméra" alt="Activer ou désactiver la caméra"
                class="w-24 h-24 justify-center bg-gradient-to-tr from-blue-950 to-blue-950/70" rounded variant="ghost">
                <UIcon :name="isCameraOpen ? 'i-lucide-camera-off' : 'i-lucide-camera'" class="w-10 h-10" />
            </UButton>
        </div>
        <qrcode-capture :formats @detect="onDetect" ref="fileInput" class="hidden"></qrcode-capture>
    </div>
</template>


<script setup>
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'
const fileInput = ref(null)
const formats = ['ean_13', 'ean_8', 'upc_a', 'upc_e', 'code_39', 'code_128', 'code_93', 'itf', 'qr_code']

const buttons = [
    { icon: 'i-lucide-history', title: 'Historique', to: '/historique' },
    { icon: 'i-lucide-upload', title: 'Importer un fichier' },
    { icon: 'i-lucide-camera', title: 'Activer / déscativer la caméra' }
]

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