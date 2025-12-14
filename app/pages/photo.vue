<template>
  <div class="h-screen overflow-hidden p-5">
    <client-only>
      <photo-editor v-if="isEdit" />
      <div
        v-else
        class="w-full h-full flex flex-col lg:flex-row gap-5 justify-center items-center">
        <div class="w-full lg:w-3/4 h-full flex flex-col justify-center items-center lg:items-end">
          <div class="w-auto max-h-full bg-white rounded-2xl p-5 shadow">
            <div class="relative">
              <video
                ref="videoRef"
                autoplay
                playsinline
                class="w-auto max-h-full lg:max-h-100 xl:max-h-full" 
                :style="{ filter: selectedFilter[selectedFrame.image.length] }" />
              <div
                v-if="startTimer && restTime"
                class="absolute top-0 left-0 w-full h-full flex justify-center items-center text-white/50 text-7xl">
                <p>{{ restTime }}</p>
              </div>

              <div v-if="showFilterList" class="w-full absolute bottom-0 flex gap-1 justify-center items-center">
                <button
                  v-for="(filter, index) in filterList" :key="index"
                  @click="changeFilter(filter.filter)"
                  class=" cursor-pointer"
                  :class="{
                    'bg-pink-600/50': filter.filter == selectedFilter[selectedFrame.image.length],
                    'bg-black/50': filter.filter !== selectedFilter[selectedFrame.image.length],
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
            <div
              class="bg-white mt-5 gap-5 flex justify-center items-center">
              <button
                :disabled="startTimer"
                @click="changeDefaultTimer"
                class="p-3 rounded-full relative">
                <icons-timer class="size-7" />
                <div class="absolute bottom-1 right-1 w-5 h-5 bg-red-600 text-white rounded-full flex justify-center items-center text-xs">
                  <p>{{ defaultTimer }}</p>
                </div>
              </button>
              <button
                @click="capture"
                :disabled="selectedFrame.image.length >= selectedFrame.shots || startTimer"
                class="text-white bg-red-600 disabled:bg-gray-400 p-3 rounded-full">
                <icons-camera class="size-7" />
              </button>
              <button
                @click="changeFilterState"
                :disabled="startTimer"
                class="p-3 rounded-full relative">
                <icons-filter class="size-7" />
              </button>
            </div>
          </div>
        </div>
        <div class="absolute top-5 right-5 lg:hidden">
          <button
            @click="changePreviewState"
            class="text-white bg-pink-500 disabled:bg-gray-400 p-3 rounded-full relative">
            <icons-collage class="size-7"/>
            <div v-if="selectedFrame.image.length" class="w-6 h-6 absolute -top-2 -right-1 rounded-full bg-red-600 text-white flex justify-center items-center text-sm">
              {{ selectedFrame.image.length }}
            </div>
          </button>
        </div>
        <div
          @click.self="changePreviewState"
          class="w-full h-full lg:w-1/4 fixed inset-0 lg:static bg-black/50 lg:bg-black/0 flex flex-col justify-center items-center gap-5 p-5"
          :class="{
            '-z-20 lg:z-30': !showPreview
          }">
          <div class="w-full max-w-lg bg-white lg:bg-white/0 p-5 rounded-2xl flex flex-col gap-5 md:justify-center items-center overflow-y-auto">
            <p class="font-semibold text-lg">Preview</p>
            <div id="preview" class="relative">
              <img
                :src="previewImage"
                class="w-auto max-h-120 border border-gray-200" />
              <div class="absolute top-0 left-0 w-full h-full">
                <div
                  v-for="(preview, index) in previewButtonStyle" :key="index" >
                  <div
                    @click.self="changeDeleteButtonState(index)"
                    class="absolute flex justify-center items-center"
                    :style="preview">
                    <button
                      v-if="showDeleteButton && deleteButtonIndex == index"
                      @click="deleteImage(index)"
                      class="text-white rounded-full p-1 bg-black">
                      <icons-trash class="size-5"/>
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <div
              class="w-full flex items-center justify-center gap-5">
              <button
                @click="changePreviewState"
                class="bg-white border border-gray-200 lg:hidden w-full py-2.5 rounded-full">
                Close
              </button>
              <button
                @click="editPhoto"
                :disabled="selectedFrame.image.length < selectedFrame.shots"
                class="bg-red-600 disabled:bg-gray-400 text-white w-full py-2.5 rounded-full">
                Next
              </button>
            </div>
          </div>
        </div>
      </div>
    </client-only>
    <div v-if="isLoading" class="absolute top-0 left-0 w-full h-full bg-black/10 flex justify-center items-center">
      <div class="w-10 h-10 border-4 border-pink-500 border-b-pink-600/30 rounded-full animate-spin" />
    </div>
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
  const isLoading = ref<boolean>(true)
  const videoRef = ref<HTMLVideoElement | null>(null)
  let stream: MediaStream | null = null
  const errorVideo = ref<boolean>(false)
  const errorMessage = ref<string>('')
  const previewImage = ref<string>('')
  const showPreview = ref<boolean>(false)
  const startTimer = ref<boolean>(false)
  const defaultTimer = ref<number>(3)
  const restTime = ref<number>(3)
  const isEdit = ref<boolean>(false)
  const showDeleteButton = ref<boolean>(false)
  const deleteButtonIndex = ref<number>()

  const selectedFilter = ref<string[]>(['none'])
  const showFilterList = ref<boolean>(false)
  const filterList = ref<Record<string, any>[]>([
    {
      name: "no filter",
      filter: "none"
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
      name: "retro film",
      filter: "brightness(1.05) contrast(0.95) saturate(0.8) sepia(0.3)"
    },
    {
      name: "pink",
      filter: "brightness(1.1) saturate(1.3) hue-rotate(25deg)"
    }
  ])
  const videoYPosition = ref<number>(0)
  const clientWidth = ref<number>(0)
  const clientHeight = ref<number>(0)
  const selectedFrame = ref<Frame>({
    id: "basic-4",
    name: "basic 4 layout",
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
      image: null,
      customizeable: true,
      overlay: false
    },
    image: []
  })

  type PreviewStyle = {
    top: string,
    left: string,
    width: string,
    height: string
  }
  const previewButtonStyle = ref<PreviewStyle[]>([])

  onMounted(async () => {
    if (route.query.frame) {
    } else {
      router.push('/frame-selection')
    }

    try {
      stream = await navigator.mediaDevices.getUserMedia({
        video: {
          facingMode: 'user',
          // width: { ideal: 1920 },
          // height: { ideal: 1440 },
          // aspectRatio: 4 / 3
        },
        audio: false
      })

      if (videoRef.value) {
        videoRef.value.srcObject = stream
        await videoRef.value.play()
        const videoHeight = videoRef.value.clientHeight
        clientWidth.value = window.innerWidth
        clientHeight.value = window.innerHeight
        videoYPosition.value = (innerHeight - videoHeight) / 2
      }
      await convertFrameToCanvas()
      isLoading.value = false
    } catch (error: any) {
      errorVideo.value = true
      const message = error.message.toLowerCase()
      if (message.includes('permission')) {
        errorMessage.value = 'Cannot access camera. Please allow camera permissions in your browser settings and refresh the page.'
      } else {
        errorMessage.value = 'Oops.. something went wrong. Refresh the page to try again.'
      }
      isLoading.value = false
    }
  })

  watch(showPreview, (newValue) => {
    if (newValue) {
      convertFrameToCanvas()
    }
  })
  const convertFrameToCanvas = async (): Promise<void> => {
    const frame = selectedFrame.value
    const canvas = document.createElement('canvas')
    const context = canvas.getContext('2d')

    canvas.width = frame.canvas.width
    canvas.height = frame.canvas.height

    const previewWidth = document.getElementById('preview')?.clientWidth || 0

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

            const scale = Math.max(slot.width / capturedImage.width, slot.height / capturedImage.height)
            const scaleX = capturedImage.width * scale
            const scaleY = capturedImage.height * scale
            if (contextImage) {
              const xCoor = (slot.width - scaleX) / 2
              const yCoor = (slot.height - scaleY) / 2
              const filter = selectedFilter.value[x] || "none"
              contextImage.filter = filter
              contextImage.drawImage(capturedImage, xCoor, yCoor, scaleX, scaleY)
              context.drawImage(canvasImage, slot.x, slot.y, slot.width, slot.height)

              const previewStyle: PreviewStyle = {
                top: slot.y * (previewWidth / canvas.width) + 'px',
                left: slot.x * (previewWidth / canvas.width) + 'px',
                width: slot.width * (previewWidth / canvas.width) + 'px',
                height: slot.height * (previewWidth / canvas.width) + 'px'
              }
              previewButtonStyle.value[x] = previewStyle
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
    if (frame.image.length >= frame.shots) {
      showPreview.value = true
    }
  }

  const capture = () => {
    if (selectedFrame.value.image.length < selectedFrame.value.shots) {
      startTimer.value = true
      const photoInterval = setInterval(() => {
        if (restTime.value) {
          restTime.value --
          if (restTime.value == 0) {
            const audio = new Audio('/sounds/mixkit-camera-shutter-hard-click-1430.mp3')
            audio.play()
          }
        } else {
          clearInterval(photoInterval)
          if (videoRef.value) {
            const canvas = document.createElement('canvas')
            canvas.width = videoRef.value.videoWidth
            canvas.height = videoRef.value.videoHeight
            const ctx = canvas.getContext('2d')
            if (ctx) {
                ctx.drawImage(videoRef.value, 0, 0, canvas.width, canvas.height)
                selectedFrame.value.image.push(canvas)
                convertFrameToCanvas()
            }
          }
          const prevIndex = selectedFrame.value.image.length - 1
          const prevFilter = selectedFilter.value[prevIndex] || "none"
          selectedFilter.value[prevIndex + 1] = prevFilter
          startTimer.value = false
          restTime.value = defaultTimer.value
        }
      }, 1000)
    }
  }
  const changeDeleteButtonState = (index?: number) => {
    if (index !== undefined && showDeleteButton.value && index == deleteButtonIndex.value) {
      showDeleteButton.value = false
      deleteButtonIndex.value = undefined
    } else {
      showDeleteButton.value = index !== undefined && index >= 0 ? true : false
      deleteButtonIndex.value = index
    }
  }

  const deleteImage = async(index: number) => {
    selectedFrame.value.image.splice(index, 1)
    selectedFilter.value.splice(index, 1)
    previewButtonStyle.value = []
    showDeleteButton.value = false
    await convertFrameToCanvas()
  }
  const changePreviewState = () => {
    const newState = !showPreview.value
    showPreview.value = newState
  }

  const editPhoto = () => {
    isEdit.value = true
  }
  const changeDefaultTimer = () => {
    if (defaultTimer.value == 3) {
      defaultTimer.value = 5
    } else if (defaultTimer.value == 5) {
      defaultTimer.value = 10
    } else {
      defaultTimer.value = 3
    }
    restTime.value = defaultTimer.value
  }
  const changeFilterState = () => {
    const newValue = !showFilterList.value
    showFilterList.value = newValue
  }

  const changeFilter = (filter: string) => {
    const currentIndex = selectedFrame.value.image.length
    selectedFilter.value[currentIndex] = filter
  }

</script>

<style scoped>
  video {
    transform: scaleX(-1);
  }
</style>