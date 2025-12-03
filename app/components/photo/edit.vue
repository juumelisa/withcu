<template>
  <div class="flex flex-col lg:flex-row gap-5">
    <div class="w-full lg:w-1/3 xl:w-2/4 flex">
      <div class="w-20 flex flex-col gap-3 bg-blue-50">
        <button
          v-for="menu in menuList"
          :key="menu.name"
          @click="changeActiveMenu(menu.name)"
          class="w-full flex flex-col items-center justify-center"
          :class="{
            'text-blue-400': menu.name == activeTab
          }">
          <component :is="menu.icon" class="size-9"/>
          <p class="capitalize">{{ menu.name }}</p>
        </button>
      </div>
      <div class="w-full bg-white border-r border-gray-200 h-dvh overflow-hidden">
        <layout-option
          v-if="activeTab === 'layout'"
          v-model:layout="currentLayout" />
        <frame-option
          v-if="activeTab === 'frame'"
          v-model:current-frame="currentFrame"
          :current-layout="currentLayout" />
      </div>
    </div>
    <div class="w-full lg:w-1/3 xl:w-2/4 flex">
      <div class="w-full p-5 flex justify-center relative">
        <div class="w-auto relative">
          <img :src="printedImage" class="w-50 h-auto border border-gray-200"/>
          <div class="w-full h-full absolute top-0 left-0 px-2 flex flex-col">
            <div v-if="currentLayout.header" class="w-full h-21 group relative">
              <!-- <div class="w-full h-full hidden group-hover:block relative">
                <button class="absolute top-2 right-0 text-green-600">
                  <icons-edit class="size-6"/>
                </button>
              </div> -->
            </div>
            <div class="flex flex-col gap-2 h-full">
              <div v-for="(images, index) in selectedImages" :key="index" class="w-full h-30 group">
                <div class="w-full h-2"></div>
                <button
                  @click="deleteImage(index)" class="hidden group-hover:bg-black/10 group-hover:flex justify-center items-center w-full h-full">
                  <icons-trash class="size-6 text-white"/>
                </button>
              </div>
            </div>
            <div class="w-full"></div>
          </div>

        </div>
        <div class="absolute right-0 bottom-10">
          <button
            class="p-3 flex items-center justify-center gap-2 bg-white border border-gray-200 rounded-full mb-3">
            <icons-camera class="size-6"/>
          </button>
          <button
            @click="saveImage"
            class="p-3 flex items-center justify-center gap-2 bg-linear-to-r from-pink-100 to-blue-300 text-blue-900 outline-0 rounded-full">
            <icons-save class="size-6"/>
          </button>
        </div>
      </div>
    </div>
    <div class="w-full lg:w-1/3 xl:w-1/4 p-5 bg-white">
      <h2 class="mb-3 font-semibold text-lg">Your gallery</h2>
      <div class="w-full grid grid-cols-1 lg:grid-cols-2 gap-3">
        <div
          v-for="(image, index) in imageUrls"
          :key="index"
          @click="selectImage(index)"
          class="relative">
          <img :src="image" class="w-full object-cover"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import Collage from '../icons/collage.vue';
import Frame from '../icons/frame.vue';
import Text from '../icons/text.vue';

type Props = {
  imageUrls: string[],
  photoCanvases: HTMLCanvasElement[]
}
type SelectedImage = HTMLCanvasElement | null
const props = defineProps<Props>()
const selectedImages = ref<SelectedImage[]>([])
const frameImage = ref<HTMLCanvasElement | null>()
const printedImage = ref<string>('')
const activeTab = ref<string>("layout")
const currentDate = ref<string>(new Date().toLocaleDateString())
const currentLayout = ref<any>({
  header: true,
  footer: true,
  strip: 3,
  vertical: true
})

type Frame = {
  type: string,
  color?: string
}
const currentFrame = ref<Frame>({
  type: "basic",
  color: "white"
})

const menuList = ref([
  {
    name: "layout",
    icon: Collage,
  },
  {
    name: "frame",
    icon: Frame
  },
  {
    name: "text",
    icon: Text
  }
])
onMounted( () => {
  generateImage()
})

watch(currentLayout, () => {
  const strip = currentLayout.value.strip
  const newImages = selectedImages.value.filter((el, idx) => idx < strip)
  selectedImages.value = newImages
  generateImage()
})

watch(currentFrame, () => {
  generateImage()
})

const changeActiveMenu = (menu: string) => {
  activeTab.value = menu
}

const generateImage = async() => {
  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')
  
  canvas.width = 600
  canvas.height = 1800
  const padding = 24
  const width = 552
  const height = 368
  const top = currentLayout.value.header ? 216 : 0
  if (context) {
    if (frameImage.value) {
      context.drawImage(frameImage.value, 0, 0, canvas.width, canvas.height)
    } else {
      // const gradient = context.createLinearGradient(0, 0, 0, canvas.height)
      // gradient.addColorStop(0, "#bfdbfe")
      // gradient.addColorStop(0.25, "#93c5fd")
      // gradient.addColorStop(0.5, "#fdf2f8")
      // gradient.addColorStop(0.75, "#fce7f3")
      // gradient.addColorStop(1, "#fbcfe8") 
      context.fillStyle = currentFrame.value.color ?? "white" // or color
      context.fillRect(0, 0, canvas.width, canvas.height)
    }
    for (let x = 0; x < currentLayout.value.strip; x++) {
      const yCoordinate = top + (x * height) + (padding * (x + 1))
      if (selectedImages.value[x]) {
        // if (selectedFilter.value[index]) {
        //   context.filter = selectedFilter.value[index]
        // } else {
        //   context.filter = "none"
        // }
        const image = selectedImages.value[x]
        if (image) {
          const scaleY = image.height * (width / image.width)
          const scaleX = image.width * (height / image.height)
          context.drawImage(image, padding, yCoordinate, scaleX, height)
        }

      } else {
        context.fillStyle = '#F3F3F3'
        context.fillRect(padding, yCoordinate, width, height)
      }
    }


    if (!frameImage.value) {
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
  }
  const dataUrl = canvas.toDataURL('image/png')
  printedImage.value = dataUrl
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

const selectImage = (index: number) => {
  if (props.photoCanvases[index] && selectedImages.value.length < currentLayout.value.strip) {
    selectedImages.value.push(props.photoCanvases[index])
    generateImage()
  } 
}

const deleteImage = (index: number) => {
  const newImages = selectedImages.value.filter((el, idx) => idx != index)
  selectedImages.value = newImages
  generateImage()
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

</script>