<template>
  <div class="h-full overflow-y-auto p-5">
    <p>Frame</p>
    <div class="grid grid-cols-3 gap-3 my-3">
      <button
        v-for="frame in frameOptionList"
        :key="frame"
        @click="changeCurrentFrame(frame)"
        class="w-full flex justify-center rounded-full py-3 capitalize"
        :class="{
          'bg-white border border-gray-200': frame !== currentFrame,
          'bg-linear-to-r from-pink-100 to-blue-300': frame === currentFrame
        }">
        {{ frame }}
      </button>
    </div>
    <div>
      <div v-if="currentFrame === 'basic'" class="w-full grid grid-cols-2 md:grid-cols-3 gap-2">
        <button
          v-for="(color, index) in frameBasic"
          :key="color"
          @click="changeFrame(color)"
          class="flex flex-col justify-center items-center border border-gray-200 p-5">
          <img
            v-if="frameBasicImages[index]"
            :src="frameBasicImages[index]"
            class="w-full"
            :class="{
              'border border-gray-200': color === 'white'
            }" />
          <p class="text-xs">{{color}}</p>
        </button>
        <div class="w-full h-full flex items-center justify-center relative">
          <icons-edit />
          <input
            type="color"
            v-model="customColor"
            class="w-full h-full absolute top-0 left-0 opacity-0" />
        </div>
      </div>
      <div v-else-if="currentFrame === 'custom'">
        <div class="w-full min-h-80 border border-dashed border-gray-200 p-5 flex flex-col justify-center items-center text-gray-500 relative">
          <icons-save class="size-6 rotate-180" />
          <p>Upload your file here</p>
          <input
            type="file"
            accept=".png"
            @change="onFileChange"
            class="absolute top-0 left-0 w-full h-full opacity-0" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
type Frame = {
  type: string,
  color?: string,
  image?: HTMLCanvasElement
}
type Props = {
  currentFrame: Frame,
  currentLayout: any
}

const props = defineProps<Props>()
const customColor = ref<string>()

const frameOptionList = ['basic', 'custom', 'other']
const currentFrame = ref<string>('basic')

const frameBasic = ref<string[]>([
  'white', 'Pink', 'LightBlue', 'Lavender', 'DarkTurquoise'
])

const frameBasicImages = ref<string[]>([])
const emits = defineEmits<{
  (e: 'update:currentFrame', value: Frame): void
}>()
onMounted(() => {
  frameBasic.value.forEach((color, index) => {
    generateImageBasic(index)
  })
})

watch(() => props.currentLayout, () => {
  frameBasic.value.forEach((color, index) => {
    generateImageBasic(index)
  })
})

// watch(customColor, (newValue) => {
//   if (newValue) {
//     if (!frameBasic.value.find(el => el == newValue)) {
//       frameBasic.value.push(newValue)
//     }
//     frameBasic.value.forEach((color, index) => {
//       generateImageBasic(index)
//     })
//   }
// })

const generateImageBasic = async (index: number) => {
  const color = frameBasic.value[index]
  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')
  
  canvas.width = 600
  canvas.height = 1800
  const padding = 24
  const width = 552
  const height = 368
  const top = props.currentLayout.header ? 216 : 0
  if (context) {
      // const gradient = context.createLinearGradient(0, 0, 0, canvas.height)
      // gradient.addColorStop(0, "#bfdbfe")
      // gradient.addColorStop(0.25, "#93c5fd")
      // gradient.addColorStop(0.5, "#fdf2f8")
      // gradient.addColorStop(0.75, "#fce7f3")
      // gradient.addColorStop(1, "#fbcfe8") 
    // if (cu)
    context.fillStyle = color ?? 'white' // or color
    context.fillRect(0, 0, canvas.width, canvas.height)
    for (let x = 0; x < props.currentLayout.strip; x++) {
      const yCoordinate = top + (x * height) + (padding * (x + 1))
      context.fillStyle = '#F3F3F3'
      context.fillRect(padding, yCoordinate, width, height)
    }

  }
  const dataUrl = canvas.toDataURL('image/png')
  frameBasicImages.value[index] = dataUrl
}

const changeFrame = (color?: string, image?: string) => {
  console.log(color)
  emits('update:currentFrame', {
    color,
    type: "basic"
  })
}
const changeCurrentFrame = (frame: string) => {
  currentFrame.value = frame
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

const onFileChange = async(e: Event) => {
  const target = e.target as HTMLInputElement
  const file = target.files?.[0]
  if (file) {
    const imageUrl = URL.createObjectURL(file)
    const canvasImage = await imageToCanvas(imageUrl)
    emits('update:currentFrame', {
      color: '',
      type: 'custom',
      image: canvasImage
    })
  }
}
</script>