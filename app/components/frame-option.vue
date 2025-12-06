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
      <div v-if="currentFrame === 'basic'" class="w-full grid grid-cols-5 md:grid-cols-7 xl:grid-cols-9 gap-2">
        <button
          v-for="(color, index) in frameBasic"
          :key="color"
          @click="changeFrame(color)"
          class="cursor-pointer w-10 h-10 rounded-full"
          :class="{
            'border border-gray-200': currentFrameDetail.color !== color,
            'border-3 border-red-600': currentFrameDetail.color === color
          }"
          :style="{ backgroundColor: color }" />
        <div
          class="cursor-pointer w-10 h-10 rounded-full border border-gray-200 flex justify-center items-center">
          <icons-edit />
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
  currentFrameDetail: Frame,
  currentLayout: any
}

const props = defineProps<Props>()
const customColor = ref<string>()

const frameOptionList = ['basic', 'custom', 'other']
const currentFrame = ref<string>('basic')

const frameBasic = ref<string[]>([
  'white', 'whitesmoke',
  'ivory', 'lemonchiffon', 'lightgoldenrodyellow', 'palegoldenrod',
  'aliceblue', 'azure', 'lightblue', 'skyblue', 'lavenderblush',
  'lightpink', 'Pink', 'lightcoral',
  'tomato', 'orangered', 'red',
  'Lavender', 'thistle', 'palevioletred',
  'aquamarine', 'lightgreen', 'limegreen', 'DarkTurquoise', 'seagreen', 'teal', 'olive',
  'slateblue', 'mediumblue', 'navy', 'midnightblue',
  'slategray', 'gray', 'dimgray', 'darkslategray',
  'black'
])

const frameBasicImages = ref<string[]>([])
const emits = defineEmits<{
  (e: 'update:currentFrameDetail', value: Frame): void
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
  emits('update:currentFrameDetail', {
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
    emits('update:currentFrameDetail', {
      color: '',
      type: 'custom',
      image: canvasImage
    })
  }
}
</script>