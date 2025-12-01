
<template>
  <div class="w-full flex flex-col xl:flex-row xl:justify-between gap-5 p-5">
    <div class="xl:w-1/5" />
    <div class="w-full lg:w-auto flex flex-col items-center">
      <div class="w-full flex flex-col">
        <div class="w-full">
          <div class="relative flex flex-col justify-center items-center">
            <video
              ref="videoRef"
              autoplay
              playsinline
              class="max-w-full"
              :style="{ filter: selectedFilter[photoCanvases.length] }" />
            <div v-if="startTimer" class="absolute top-0 left-0 w-full h-full flex items-center justify-center opacity-30">
              <p class="text-white text-7xl">{{ restTime }}</p>
            </div>
            <div v-if="showFilterList" class="absolute bottom-0 flex gap-1">
              <button
                v-for="(filter, index) in filterList" :key="index"
                @click="changeFilter(filter.filter)"
                class=" cursor-pointer"
                :class="{
                  'bg-pink-600/50': filter.filter == selectedFilter[photoCanvases.length],
                  'bg-black/50': filter.filter !== selectedFilter[photoCanvases.length],
                }">
                <div class="relative w-15 h-15">
                  <img
                    src="https://res.cloudinary.com/dme13qwgd/image/upload/v1752902754/samples/people/boy-snow-hoodie.jpg"
                    class="h-full object-cover"
                    :style="{ filter: filter.filter }"/>
                </div>
                <p class="text-white text-[8px] capitalize text-center">{{ filter.name }}</p>
              </button>
              <button class="bg-black/50 text-white text-[8px]">
                <div class="w-15 h-15 flex justify-center items-center">
                  <icons-next />
                </div>
                <p>More</p>
              </button>
            </div>
          </div>
          <p v-if="errorMessage" class="text-red-600 text-xl text-center">{{ errorMessage }}</p>
        </div>

        <p class="text-white">{{selectedFilter[photoCanvases.length - 1]}}</p>
        <div class="w-full py-2 gap-5 flex justify-center items-center">
          <button
            @click="changeDefaultTimer"
            :disabled="startTimer"
            class="relative w-10 h-10 flex items-center justify-center cursor-pointer disabled:cursor-default">
            <icons-timer class="size-7"/>
            <div class="w-4 h-4 absolute bottom-1 right-0.5 bg-black text-white rounded-full flex justify-center items-center text-xs">{{ defaultTimer }}</div>
          </button>
          <button @click="capture" :disabled="startTimer" class="w-14 h-14 bg-red-600 rounded-full flex items-center justify-center cursor-pointer disabled:cursor-default">
            <icons-camera class="text-white size-6" />
          </button>
          <button
            @click="changeFilterState"
            :disabled="startTimer"
            class="relative w-10 h-10 flex items-center justify-center cursor-pointer">
            <icons-filter class="size-7"/>
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
            :disabled="strip === stripNumber || imageUrls.length > stripNumber"
            :class="{
              'border-pink-600 text-pink-600': stripNumber == strip,
              'border-gray-200': stripNumber !== strip
            }">
            {{ strip }} strip photo
          </button>
        </div>
      </div>
    </div>
    <div class="w-full xl:w-1/5 flex flex-col justify-center items-center relative">
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
const showFilterList = ref<boolean>(false)

const filterList = ref<Record<string, any>[]>([
  {
    name: "no filter",
    filter: ""
  },
  {
    name: "bright clean",
    filter: 'brightness(1.15) contrast(1.1) saturate(1.1)'
  },
  {
    name: "warm wedding",
    filter: 'brightness(1.1) contrast(1.05) saturate(1.25) sepia(0.15)'
  },
  {
    name: "teal orange",
    filter: 'contrast(1.2) saturate(1.4) sepia(0.25) hue-rotate(-15deg)'
  },
  {
    name: "BW",
    filter: 'grayscale(1) contrast(1.2) brightness(1.1)'
  },
  {
    name: "pastel",
    filter: 'brightness(1.25) saturation(1.15) contrast(0.9) hue-rotate(15deg)'
  },
  {
    name: "retro film",
    filter: "brightness(1.05) contrast(0.95) saturate(0.8) sepia(0.3)"
  },
  {
    name: "pink",
    filter: "brightness(1.1) saturate(1.3) hue-rotate(25deg)"
  }
])

const FILTER_BRIGHT_CLEAN = "brightness(1.15) contrast(1.1) saturate(1.1)";
const FILTER_WARM_WEDDING = "brightness(1.1) contrast(1.05) saturate(1.25) sepia(0.15)";
const FILTER_TEAL_ORANGE = "contrast(1.2) saturate(1.4) sepia(0.25) hue-rotate(-15deg)";
const FILTER_BW = "grayscale(1) contrast(1.2) brightness(1.1)";
const FILTER_PASTEL = "brightness(1.25) saturation(1.15) contrast(0.9) hue-rotate(15deg)";
const FILTER_RETRO_FILM = "brightness(1.05) contrast(0.95) saturate(0.8) sepia(0.3)";
const FILTER_PINK = "brightness(1.1) saturate(1.3) hue-rotate(25deg)";
const FILTER_GLAM = "brightness(1.15) contrast(0.95) saturate(1.1) blur(1px)";
const IG_CLARENDON = "brightness(1.1) contrast(1.3) saturate(1.35)";
const IG_VALENCIA = "brightness(1.08) contrast(0.9) saturate(1.1) sepia(0.08)";
const IG_JUNO = "brightness(1.12) contrast(1.15) saturate(1.4) hue-rotate(-5deg)";
const IG_LARK = "brightness(1.25) contrast(1.1) saturate(0.9)";
const IG_GINGHAM = "brightness(1.05) contrast(0.9) sepia(0.15)";
const IG_ADEN = "brightness(1.1) contrast(0.9) saturate(0.85) sepia(0.1) hue-rotate(10deg)";
const IG_REYES = "brightness(1.25) contrast(0.9) saturate(0.75)";
const IG_WILLOW = "grayscale(1) brightness(1.1) contrast(0.9) sepia(0.1)";
const IG_SLUMBER = "brightness(1.25) contrast(0.8) saturate(0.8) sepia(0.25)";
const IG_CREMA = "brightness(1.1) contrast(0.9) saturate(0.9) sepia(0.05)";

const selectedFilter = ref<string[]>([''])
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

const changeFilterState = () => {
  const newValue = !showFilterList.value
  showFilterList.value = newValue
}

const changeDefaultTimer = () => {
  const availableTimer = [3, 5, 10]
  const currentTimerIndex = availableTimer.findIndex(el => el == defaultTimer.value)
  const newTime = availableTimer[currentTimerIndex + 1]
  defaultTimer.value = newTime ? newTime : 3
  restTime.value = defaultTimer.value
}

const changeFilter = (filter: string) => {
  const currentIndex = photoCanvases.value.length
  selectedFilter.value[currentIndex] = filter
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
            const newIndex = photoCanvases.value.length
            if (!selectedFilter.value[newIndex] && selectedFilter.value[newIndex - 1]) {
              selectedFilter.value[newIndex] = selectedFilter.value[newIndex - 1] || ''
            }

            if (selectedFilter.value[newIndex]) {
              selectedFilter.value.push(selectedFilter.value[newIndex])
            } else {
              selectedFilter.value.push('')
            }
            // const currentFilter = selectedFilter.value[newIndex] ? selectedFilter.value[newIndex] : ''
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
        if (selectedFilter.value[index]) {
          context.filter = selectedFilter.value[index]
        } else {
          context.filter = "none"
        }
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
  const newFilter = selectedFilter.value.filter((el, idx) => index != idx)
  imageUrls.value = newImageArr
  photoCanvases.value = newCanvasImage
  selectedFilter.value = newFilter
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
