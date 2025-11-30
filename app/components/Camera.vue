
<template>
  <div class="w-full grid grid-cols-1 md:grid-cols-2 gap-5 p-5">
    <div class="w-full flex flex-col items-center">
      <div class="relative">
        <video ref="videoRef" autoplay playsinline class="max-w-full rounded-lg" />
        <canvas
          ref="photoCanvas"
          class="hidden" />
        <div v-if="startTimer" class="absolute top-0 left-0 w-full h-full flex items-center justify-center opacity-30">
          <p class="text-white text-7xl">{{ restTime }}</p>
        </div>
      </div>
      <p v-if="errorMessage" class="text-red-600 text-xl text-center">{{ errorMessage }}</p>
      <div class="w-full grid grid-cols-1 md:grid-cols-2 gap-3 mt-3">
        <button
          v-for="strip in [3, 4]"
          :key="strip"
          @click="stripNumber = strip"
          class="w-full p-3 flex justify-center items-center border rounded cursor-pointer disabled:cursor-default"
          :disabled="strip === stripNumber"
          :class="{
            'border-pink-600 text-pink-600': stripNumber == strip,
            'border-gray-200': stripNumber !== strip
          }">
          {{ strip }} strip photo
        </button>
      </div>
      <button
        @click="capture"
        :disabled="imageUrls.length >= stripNumber"
        class="w-full bg-linear-to-r from-pink-100 to-blue-300 outline-0 rounded p-3 mt-5 disabled:cursor-not-allowed">
        Take Photo
      </button>
    </div>
    <div class="flex justify-center items-center">
      <div>
        <h2 class="mb-3">Preview</h2>
        <div class="w-50 h-150 border border-gray-200 flex flex-col gap-2 p-2 bg-white">
          <div v-for="index in stripNumber" :key="index" class="w-full group relative">
            <img v-if="imageUrls[index - 1]" :src="imageUrls[index - 1]" class="w-full h-32 object-cover" />
            <div v-else class="w-full border border-gray-200 h-32 flex justify-center items-center bg-black/80">
              <p class="italic text-gray-400">Your photo here</p>
            </div>
            <div v-if="imageUrls[index - 1]" class="hidden group-hover:block absolute top-0 left-0 bg-black/20 w-full h-full">
              <button class="absolute top-2 right-2 bg-gray-100 rounded-full p-1">
                <icons-trash class="size-5" />
              </button>
            </div>
          </div>
          <div class="w-48 italic h-full flex flex-col justify-center items-center">
            <p>{{ currentDate }}</p>
            <p class="text-[10px]">www.withcu.xyz</p>
          </div>
        </div>
      </div>
    </div>
    <div class="fixed bottom-5 right-5 flex items-center gap-3">
      <button class="py-3 px-5 flex items-center justify-center gap-2 border border-pink-100 outline-0 rounded-full">
        <icons-edit class="size-5" />
        <p>Edit</p>
      </button>
      <button class="py-3 px-5 flex items-center justify-center gap-2 bg-linear-to-r from-pink-100 to-blue-300 outline-0 rounded-full">
        <icons-print class="size-5" />
        <p>Print</p>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
const videoRef = ref<HTMLVideoElement | null>(null)
const photoCanvas = ref<HTMLCanvasElement | null>(null)
let stream: MediaStream | null = null
const stripNumber = ref<number>(4)
const imageUrls = ref<string[]>([])
const errorVideo = ref<boolean>(false)
const errorMessage = ref<string>('')
const startTimer = ref<boolean>(false)
const restTime = ref<number>(5)
const currentDate = ref<string>(new Date().toLocaleDateString())
onMounted(async () => {
  try {
    stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'user' } // front camera
    })

    if (videoRef.value) {
      videoRef.value.srcObject = stream
      await videoRef.value.play()
    }
  } catch (error: any) {
    errorVideo.value = true
    const message = error.message.toLowerCase()
    if (message.includes('permission')) {
      errorMessage.value = 'Cannot access camera. Please allow camera permissions in your browser settings and refresh the page.'
    } else {
      errorMessage.value = 'Oops.. something went wrong. Refresh the page to try again.'
    }
  }
})

onBeforeUnmount(() => {
  if (stream) {
    stream.getTracks().forEach(track => track.stop())
  }
})

watch(imageUrls, (newValue) => {
  const imageValues = JSON.stringify(newValue)
  sessionStorage.setItem('photoboothImages', imageValues)
}, { deep: true })

const capture = () => {
  startTimer.value = true
  const photoInterval = setInterval(() => {
    if (restTime.value) {
      restTime.value --
    } else {
      clearInterval(photoInterval)
      if (videoRef.value && photoCanvas.value) {
        const canvas = photoCanvas.value
        canvas.width = videoRef.value.videoWidth
        canvas.height = videoRef.value.videoHeight
        const ctx = canvas.getContext('2d')
        if (ctx) {
            ctx.drawImage(videoRef.value, 0, 0, canvas.width, canvas.height)
            const dataUrl = canvas.toDataURL('image/png')
            imageUrls.value.push(dataUrl)
        }
      }
      startTimer.value = false
      restTime.value = 5
    }
  }, 1000)
}
</script>


<style scoped>
video {
  transform: scaleX(-1);
}
</style>
