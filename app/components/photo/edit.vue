<template>
  <div class="flex flex-col lg:flex-row gap-5">
    <div class="w-full lg:w-2/3 xl:w-3/4 flex">
      <div class="w-20">
        <layout-option
          v-model:layout="currentLayout" />
      </div>
      <div class="w-full p-5 flex justify-center">
        <img :src="printedImage" class="w-50 h-auto border border-gray-200"/>
      </div>
    </div>
    <div class="w-full lg:w-1/3 xl:w-1/4 p-5 bg-white">
      <h2 class="mb-3 font-semibold text-lg">Your gallery</h2>
      <div class="w-full grid grid-cols-1 lg:grid-cols-2 gap-3">
        <div
          v-for="(image, index) in imageUrls"
          :key="index"
          @click="selectImage(index)"
          class="relative w-full">
          <img :src="image" class="w-full object-cover"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
type Props = {
  imageUrls: string[],
  photoCanvases: HTMLCanvasElement[]
}

const props = defineProps<Props>()
const selectedImages = ref<HTMLCanvasElement[]>([])
const frameImage = ref<HTMLCanvasElement | null>()
const printedImage = ref<string>('')
const currentDate = ref<string>(new Date().toLocaleDateString())
const currentLayout = ref<any>({
  header: true,
  footer: true,
  strip: 3,
  vertical: true
})

onMounted( () => {
  generateImage()
})

watch(currentLayout, () => {
  generateImage()
})
// watch(
//   () => props.imageUrls,
//   (newValue) => {
//     console.log(newValue)
//     Object.values(newValue).forEach(async (value, index) => {
//       console.log(value)
//       const res = await imageToCanvas(value)
//       selectedImages.value[index] = res
//     })
//     generateImage()
//   },
//   {deep: true}
// )


const generateImage = async() => {
  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')
  
  canvas.width = 600
  canvas.height = 1800
  const padding = 24
  const width = 46 * 12
  const height = 368
  const top = currentLayout.value.header ? 218 : 0
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
      context.fillStyle = 'white' // or color
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
          context.drawImage(image, padding, yCoordinate, width, scaleY)
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
  if (props.photoCanvases[index]) {
    selectedImages.value.push(props.photoCanvases[index])
    generateImage()
  } 
}
</script>