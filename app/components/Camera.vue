
<template>
  <div class="w-full grid grid-cols-1 md:grid-cols-2 gap-5 p-5">
    <div class="w-full flex flex-col items-center">
      <div class="flex">
        <div class="w-full">
          <div class="relative">
            <video ref="videoRef" autoplay playsinline class="max-w-full" />
            <div v-if="startTimer" class="absolute top-0 left-0 w-full h-full flex items-center justify-center opacity-30">
              <p class="text-white text-7xl">{{ restTime }}</p>
            </div>
          </div>
          <p v-if="errorMessage" class="text-red-600 text-xl text-center">{{ errorMessage }}</p>
        </div>
        <div class="bg-blue-100 text-blue-900">
          <button
            @click="changeDefaultTimer"
            :disabled="startTimer"
            class="relative w-10 h-10 flex items-center justify-center cursor-pointer">
            <icons-timer class="size-7"/>
            <div class="w-4 h-4 absolute bottom-1 right-0.5 bg-black text-white rounded-full flex justify-center items-center text-xs">{{ defaultTimer }}</div>
          </button>
        </div>
      </div>
      <div class="w-full">
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
    </div>
    <div class="w-full flex flex-col justify-center items-center relative">
      <h2 class="mb-3">Preview</h2>
      <div class="relative border border-gray-200">
        <div v-if="printedImage">
          <img :src="printedImage" class="w-50"/>
        </div>
        <div class="absolute top-0 left-0 w-50 h-150 flex flex-col gap-2 p-2 bg-white/0">
          <div v-if="stripNumber == 3" class="h-16" />
          <div
            v-for="index in stripNumber" :key="index" class="w-full h-32 group relative"
            :class="{
              'border border-gray-200': !imageUrls[index - 1]
            }">
            <!-- <img v-if="imageUrls[index - 1]" :src="imageUrls[index - 1]" class="w-full h-32 object-cover" />
            <div v-else class="w-full border border-gray-200 h-32 flex justify-center items-center">
              <p class="italic text-gray-400">Your photo here</p>
            </div> -->
            <div v-if="imageUrls[index - 1]" class="hidden group-hover:block absolute top-0 left-0 bg-black/20 w-full h-full">
              <button
                @click="deleteImage(index - 1)"
                class="absolute top-2 right-2 bg-gray-100 rounded-full p-1">
                <icons-trash class="size-5" />
              </button>
            </div>
          </div>
        </div>
      </div>
      <div class="absolute top-10 right-0 w-10">
        <!-- <layout-option v-model:strip="stripNumber"/> -->
        <!-- <button class="border border-gray-200">
          <icons-background class="size-10"/>
        </button> -->
        <background-option
          @on-change-background="onChangeBackground"
          :background-image="printedImage" />
        <button class="border border-gray-200">
          <icons-header class="size-10"/>
        </button>
        <footer-option
          @on-change-footer="onChangeFooter"
          :background-image="printedImage" />
      </div>
    </div>
    <div class="fixed bottom-5 right-5 flex items-center gap-3">
      <button
        @click="saveImage"
        class="py-3 px-5 flex items-center justify-center gap-2 bg-linear-to-r from-pink-100 to-blue-300 outline-0 rounded-full">
        <icons-next class="size-5" />
        <p>Next</p>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
const videoRef = ref<HTMLVideoElement | null>(null)
let stream: MediaStream | null = null
const stripNumber = ref<number>(3)
const photoCanvases = ref<HTMLCanvasElement[]>([])
const imageUrls = ref<string[]>([])
const errorVideo = ref<boolean>(false)
const errorMessage = ref<string>('')
const startTimer = ref<boolean>(false)
const defaultTimer = ref<number>(5)
const restTime = ref<number>(5)
const currentDate = ref<string>(new Date().toLocaleDateString())
const printedImage = ref<string>()
const backgroundImage = ref<HTMLCanvasElement>()
const footerImage = ref<HTMLCanvasElement>()

onMounted(async () => {
  try {
    stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'user' } // front camera
    })

    if (videoRef.value) {
      videoRef.value.srcObject = stream
      await videoRef.value.play()
    }
    generateImage()
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
  generateImage()
}, { deep: true })

watch(stripNumber, () => {
  generateImage()
})

const changeDefaultTimer = () => {
  const availableTimer = [3, 5, 10]
  const currentTimerIndex = availableTimer.findIndex(el => el == defaultTimer.value)
  const newTime = availableTimer[currentTimerIndex + 1]
  defaultTimer.value = newTime ? newTime : 3
  restTime.value = defaultTimer.value
}

const capture = () => {
  startTimer.value = true
  const photoInterval = setInterval(() => {
    if (restTime.value) {
      restTime.value --
    } else {
      clearInterval(photoInterval)
      if (videoRef.value) {
        const canvas = document.createElement('canvas')
        canvas.width = videoRef.value.videoWidth
        canvas.height = videoRef.value.videoHeight
        const ctx = canvas.getContext('2d')
        if (ctx) {
            ctx.drawImage(videoRef.value, 0, 0, canvas.width, canvas.height)
            const dataUrl = canvas.toDataURL('image/png')
            imageUrls.value.push(dataUrl)
        }
        photoCanvases.value.push(canvas)
      }
      startTimer.value = false
      restTime.value = defaultTimer.value
    }
  }, 1000)
}

const generateImage = async() => {
  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')
  
  canvas.width = 600
  canvas.height = 1800
  const padding = 24
  const width = 46 * 12
  const height = 32 * 12
  const top = stripNumber.value == 3 ? 218 : 0
  if (context) {
    if (backgroundImage.value) {
      context.drawImage(backgroundImage.value, 0, 0, canvas.width, canvas.height)
    } else {
      // const gradient = context.createLinearGradient(0, 0, 0, canvas.height)
      // gradient.addColorStop(0, "#bfdbfe")
      // gradient.addColorStop(0.25, "#93c5fd")
      // gradient.addColorStop(0.5, "#fdf2f8")
      // gradient.addColorStop(0.75, "#fce7f3")
      // gradient.addColorStop(1, "#fbcfe8") 
      context.fillStyle = 'white' // or color
      context.fillRect(0, 0, canvas.width, canvas.height)
    }

    if (photoCanvases.value.length) {
      photoCanvases.value.forEach((image, index) => {
        const yCoordinate = top + (index * height) + (padding * (index + 1))
        context.drawImage(image, padding, yCoordinate, width, height)
      })
    }


    if (!backgroundImage.value) {
      await document.fonts.load("100px 'Lavishly Yours'")
      await document.fonts.ready
      if (top) {
        context.font = "normal 100px 'Lavishly Yours', cursive"
        context.fillStyle = "#000"
        context.textAlign = "center"
        context.fillText('Withcu', canvas.width / 2, padding * 4.5)

        context.font = "normal 80px 'Lavishly Yours', cursive"
        context.fillStyle = "#000"
        context.fillText('photobooth', canvas.width / 2, padding * 7.5)
      }

      const fromBottom = top ? 80 : 0
      context.font = "normal 100px 'Lavishly Yours', cursive"
      context.fillStyle = "#000"
      context.textAlign = "left"
      context.fillText('Our Memories', padding, canvas.height - (padding * 3.6 + fromBottom))

      context.font = "normal 45px 'Lavishly Yours', cursive"
      context.fillStyle = "#000"
      context.fillText(currentDate.value, padding, canvas.height - (padding + fromBottom))
    }
    if (footerImage.value) {
      const width = canvas.width
      console.log(footerImage.value.width, width)
      const height = footerImage.value.height * (width / footerImage.value.width)
      context.drawImage(footerImage.value, padding, canvas.height - height, width, height)
    }
  }
  const dataUrl = canvas.toDataURL('image/png')
  printedImage.value = dataUrl
}

const next = () => {
  console.log('next')
}

const deleteImage = (index: number) => {
  const newImageArr = imageUrls.value.filter((el, idx) => index != idx)
  const newCanvasImage = photoCanvases.value.filter((el, idx) => index != idx)
  imageUrls.value = newImageArr
  photoCanvases.value = newCanvasImage
}

const saveImage = () => {
  generateImage()
  if (printedImage.value) {
    const a = document.createElement('a')
    a.href = printedImage.value
    a.download = 'photostrip.png'
    a.click()
  }
}

const onChangeBackground = async (e: Event) => {
  const target = e.target as HTMLInputElement
  const file = target.files?.[0]
  if (file) {
    const imageUrl = URL.createObjectURL(file)
    backgroundImage.value = await imageToCanvas(imageUrl)
    generateImage()
  }
}

const onChangeFooter = async (e: Event) => {
  const target = e.target as HTMLInputElement
  const file = target.files?.[0]
  if (file) {
    const imageUrl = URL.createObjectURL(file)
    footerImage.value = await imageToCanvas(imageUrl)
    generateImage()
  }
}

const imageToCanvas = (imageSrc: string): Promise<HTMLCanvasElement> => {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.crossOrigin = 'anonymous';
    img.onload = () => {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d')!;
      canvas.width = img.width;
      canvas.height = img.height;
      ctx.drawImage(img, 0, 0);
      resolve(canvas);
    };
    img.onerror = reject;
    img.src = imageSrc;
  });
};
</script>


<style scoped>
video {
  transform: scaleX(-1);
}
</style>
