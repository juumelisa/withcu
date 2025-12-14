<template>
  <div class="w-full min-h-dvh overflow-hidden p-5 flex flex-col justify-center items-center relative">
    <h1>Choose Your Frame</h1>
    <div class="w-full max-w-6xl pt-5 overflow-hidden">
      <div class="w-full h-[450px] flex flex-col bg-white rounded-2xl p-5 overflow-hidden">
        <form-input
          name="search"
          type="text"
          placeholder="Search frame..." />
        <div class="w-full mt-10 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-5 overflow-y-auto">
          <button
            v-for="frame in frames"
            @click="selectFrame(frame)"
            :id="frame.id"
            class="flex flex-col items-center gap-3 border border-gray-200 p-5 rounded hover:bg-pink-50/50 cursor-pointer">
            <div
              class="h-full flex flex-col justify-center items-center">
              <img
                v-if="frame.image"
                :src="frame.image"
                class="max-h-40 border border-gray-200" />
            </div>
            <p class="capitalize">{{ frame.name }}</p>
          </button>
        </div>
      </div>
    </div>
    <div v-if="selectedFrame" class="w-full h-dvh fixed inset-0 bg-black/20 p-5 flex justify-center items-center">
      <div class="w-full max-w-xl h-full overflow-y-auto bg-white rounded-2xl p-5 flex flex-col justify-center items-center gap-5">
        <img
          :src="selectedFrame.image"
          class="w-auto max-h-120 border border-gray-200" />
        <div class="w-full flex flex-col md:flex-row gap-5">
          <button
            @click="removeSelectedFrame"
            class="bg-white w-full flex justify-center rounded-full py-2.5 border border-gray-200">
            Cancel
          </button>
          <button
            @click="confirmLayout"
            class="bg-blue-300 w-full flex justify-center rounded-full py-2.5">
            Use this layout
          </button>
        </div>
      </div>
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
    image: string
  }
  const selectedFrame = ref<Frame>()
  const frames = ref<Frame[]>([
    {
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
        color: 'black',
        image: null,
        customizeable: true,
        overlay: false
      },
      image: ''
    },
    {
      id: "basic-4-2",
      name: "basic 4-2 layout",
      shots: 3,
      canvas: {
        width: 1800,
        height: 1200
      },
      slots: [
        {x: 30, y: 30, width: 855, height: 555},
        {x: 915, y: 30, width: 855, height: 555},
        {x: 30, y: 605, width: 855, height: 555},
      ],
      background: {
        color: 'black',
        image: null,
        customizeable: true,
        overlay: false
      },
      image: ''
    },
    {
      id: "basic-3",
      name: "basic 3 layout",
      shots: 3,
      canvas: {
        width: 600,
        height: 1800
      },
      slots: [
        {x: 30, y: 30, width: 540, height: 360},
        {x: 30, y: 420, width: 540, height: 360},
        {x: 30, y: 810, width: 540, height: 360}
      ],
      background: {
        color: 'black',
        image: null,
        customizeable: true,
        overlay: false
      },
      image: ''
    },
    {
      id: "basic-3",
      name: "basic 3 layout (2)",
      shots: 3,
      canvas: {
        width: 600,
        height: 1800
      },
      slots: [
        {x: 30, y: 330, width: 540, height: 360},
        {x: 30, y: 720, width: 540, height: 360},
        {x: 30, y: 1110, width: 540, height: 360}
      ],
      background: {
        color: 'black',
        image: null,
        customizeable: true,
        overlay: false
      },
      image: ''
    },
    {
      id: "eid-4",
      name: "eid 4 layout",
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
        color: null,
        image: 'https://res.cloudinary.com/dme13qwgd/image/upload/v1765640447/image_copy_12_rc0qes.png',
        customizeable: true,
        overlay: true
      },
      image: ''
    },
    {
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
        color: null,
        image: 'https://res.cloudinary.com/dme13qwgd/image/upload/v1765635678/image_copy_9_fhkopu.png',
        customizeable: true,
        overlay: true
      },
      image: ''
    },
    {
      id: "christmast-4",
      name: "christmast 4 layout",
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
        color: null,
        image: 'https://res.cloudinary.com/dme13qwgd/image/upload/v1765639009/image_copy_11_mtmqhb.png',
        customizeable: true,
        overlay: true
      },
      image: ''
    },
    {
      id: "christmast-4-2",
      name: "christmast 4-2 layout",
      shots: 4,
      canvas: {
        width: 1800,
        height: 1200
      },
      slots: [
        {x: 45, y: 30, width: 840, height: 555},
        {x: 915, y: 30, width: 840, height: 555},
        {x: 45, y: 615, width: 840, height: 555},
      ],
      background: {
        color: null,
        image: "https://res.cloudinary.com/dme13qwgd/image/upload/v1765639009/image_copy_10_sl4agw.png",
        customizeable: true,
        overlay: true
      },
      image: ''
    },
  ])

  // const frameImages = ref<string[]>([])

  onMounted(async () => {
    let newFrames = frames.value.map(async (frame) => {
      const frameImage = await convertFrameToCanvas(frame)
      return {
        ...frame,
        image: frameImage
      }
    })
    for (let x = 0; x < frames.value.length; x++) {
      if (frames.value[x]) {
        const frame = frames.value[x] as Frame
        const frameImage = await convertFrameToCanvas(frame)
        frame.image = frameImage
        frames.value[x] = frame
      }
    }
    // frames.value = newFrames
  })

  const convertFrameToCanvas = async (frame: Frame): Promise<string> => {
    const canvas = document.createElement('canvas')
    const context = canvas.getContext('2d')

    canvas.width = frame.canvas.width
    canvas.height = frame.canvas.height

    if (context) {
      if (frame.background.color) {
        context.fillStyle = frame.background.color
        context.fillRect(0, 0, canvas.width, canvas.height)
      }
      frame.slots.forEach((slot) => {
        context.fillStyle = '#F3F3F3'
        context.fillRect(slot.x, slot.y, slot.width, slot.height)
      })

      if (frame.background.image && frame.background.overlay) {
        const frameImage = await imageToCanvas(frame.background.image)
        context.drawImage(frameImage, 0, 0, canvas.width, canvas.height)
      }
    }

    const dataUrl = canvas.toDataURL('image/png')
    return dataUrl
  }

  const selectFrame = (frame: Frame) => {
    selectedFrame.value = frame
  }
  const removeSelectedFrame = () => {
    selectedFrame.value = undefined
  }

  const confirmLayout = () => {
    if (selectedFrame.value) {
      useRouter().push('/photo?frame=' + selectedFrame.value.id)
    }
  }
</script>