<template>
  <div>
    <client-only>
      <div class="w-full flex flex-col xl:flex-row">
        <div class="w-1/4 hidden xl:block" />
        <div class="w-full xl:w-1/2 flex flex-col justify-center items-center">
          <div class="relative">
            <video
              ref="videoRef"
              autoplay
              playsinline
              class="max-w-full" />
            <div
              v-if="startTimer"
              class="absolute top-0 left-0 w-full h-full flex justify-center items-center text-white/50 text-7xl">
              <p>{{ restTime }}</p>
            </div>
          </div>
          <div class="w-full flex justify-center items-center mt-5">
            <button
              @click="capture"
              class="text-white bg-red-600 p-3 rounded-full">
              <icons-camera class="size-7" />
            </button>
          </div>
        </div>
        <div class="w-full xl:w-1/4 flex flex-col p-5">
          <p class="font-medium text-base">Preview</p>
          <img
            :src="previewImage"
            class="w-48" />
      </div>
      </div>
    </client-only>
  </div>
</template>

<script setup lang="ts">

type CanvasSize = {
    width: number,
    height: number
  }
  type ImageSettings = {
    x: number,
    y: number,
    width: number,
    height: number
  }
  type BackgroundSettings = {
    color: string | null,
    image: string | null,
    customizeable: boolean,
    overlay: boolean
  }
  type Frame = {
    id: string,
    name: string,
    shots: number,
    canvas: CanvasSize,
    slots: ImageSettings[],
    background: BackgroundSettings,
    image: HTMLCanvasElement[]
  }

  const route = useRoute()
  const router = useRouter()

  const videoRef = ref<HTMLVideoElement | null>(null)
  let stream: MediaStream | null = null
  const errorVideo = ref<boolean>(false)
  const errorMessage = ref<string>('')
  const previewImage = ref<string>('')

  const startTimer = ref<boolean>(false)
  const defaultTimer = ref<number>(3)
  const restTime = ref<number>(3)
  const selectedFrame = ref<Frame>({
    id: "valentine-4",
    name: "valentine 4 layout",
    shots: 4,
    canvas: {
      width: 600,
      height: 1800
    },
    slots: [
      {x: 30, y: 30, width: 540, height: 360},
      {x: 30, y: 420, width: 540, height: 360},
      {x: 30, y: 810, width: 540, height: 360},
      {x: 30, y: 1200, width: 540, height: 360}
    ],
    background: {
      color: 'white',
      image: 'https://res.cloudinary.com/dme13qwgd/image/upload/v1765635678/image_copy_9_fhkopu.png',
      customizeable: true,
      overlay: true
    },
    image: []
  })

  onMounted(async () => {
    if (route.query.frame) {
    } else {
      router.push('/frame-selection')
    }

  try {
    stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'user' } // front camera
    })

    if (videoRef.value) {
      videoRef.value.srcObject = stream
      await videoRef.value.play()
    }
    await convertFrameToCanvas()
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


  const convertFrameToCanvas = async (): Promise<void> => {
    const frame = selectedFrame.value
    const canvas = document.createElement('canvas')
    const context = canvas.getContext('2d')

    canvas.width = frame.canvas.width
    canvas.height = frame.canvas.height
    let xAxis = 0
    if (context) {
      if (frame.background.color) {
        context.fillStyle = frame.background.color
        context.fillRect(0, 0, canvas.width, canvas.height)
      }
      for (let x = 0; x < frame.shots; x++) {
        const slot = frame.slots[x]
        if (slot) {
          if (slot.x < xAxis || xAxis == 0) {
            xAxis = slot.x
          }
          xAxis = slot.x
          const capturedImage = frame.image[x]
          if (capturedImage) {
            const canvasImage = document.createElement('canvas')
            const contextImage = canvasImage.getContext('2d')

            canvasImage.width = slot.width
            canvasImage.height = slot.height

            const scaleY = capturedImage.height * (slot.width / capturedImage.width)
            if (contextImage) {
              const yCoor = (slot.height - scaleY) / 2
              contextImage.drawImage(capturedImage, 0, yCoor, slot.width, scaleY)
              context.drawImage(canvasImage, slot.x, slot.y, slot.width, slot.height)
            }
          } else {
            context.fillStyle = '#F3F3F3'
            context.fillRect(slot.x, slot.y, slot.width, slot.height)
          }
        }
      }

      if (frame.background.image && frame.background.overlay) {
        const frameImage = await imageToCanvas(frame.background.image)
        context.drawImage(frameImage, 0, 0, canvas.width, canvas.height)
      }

      context.font = "normal 24px 'Inter', Sans-serif"
      context.fillText('www.witchu.com', xAxis, frame.canvas.height - xAxis)
    }

    const dataUrl = canvas.toDataURL('image/png')
    previewImage.value = dataUrl
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
              // const newIndex = photoCanvases.value.length
              // if (!selectedFilter.value[newIndex] && selectedFilter.value[newIndex - 1]) {
              //   selectedFilter.value[newIndex] = selectedFilter.value[newIndex - 1] || ''
              // }

              // if (selectedFilter.value[newIndex]) {
              //   selectedFilter.value.push(selectedFilter.value[newIndex])
              // } else {
              //   selectedFilter.value.push('')
              // }
              // const currentFilter = selectedFilter.value[newIndex] ? selectedFilter.value[newIndex] : ''
              ctx.drawImage(videoRef.value, 0, 0, canvas.width, canvas.height)
              selectedFrame.value.image.push(canvas)
              convertFrameToCanvas()
          }
        }
        startTimer.value = false
        restTime.value = defaultTimer.value
      }
    }, 1000)
  }

</script>

<style scoped>
  video {
    transform: scaleX(-1);
  }
</style>