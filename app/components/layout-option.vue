<template>
  <div>
    <button @click="changeModalState" class="w-full cursor-pointer p-3 flex flex-col justify-center items-center">
      <icons-collage class="size-9" />
      <p>Layout</p>
    </button>
    <div
        class="fixed inset-0 z-50 flex items-center justify-center p-5 transition-all duration-300 overflow-hidden"
        :class="{
          'opacity-100 pointer-events-auto': open,
          'opacity-0 pointer-events-none': !open
        }"
      >
        <div
          @click="changeModalState"
          class="absolute inset-0 bg-black/50 transition-opacity duration-300"
          :class="{
            'opacity-100': open,
            'opacity-0': !open

          }"
        />

        <div
          class="w-full max-h-full relative z-10 bg-white rounded-xl shadow-xl transform transition-all duration-300 flex flex-col"
          :class="{
            'scale-100 opacity-100': open,
            'scale-95 opacity-0': !open
          }"
        >
          <h1 class="p-5">Change Layout</h1>
          <div class="h-full overflow-y-auto p-5">
            <div class="w-full grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-5">
              <button
                @click="changeLayout('A')"
                class="border border-gray-200 p-3 rounded flex justify-center items-center relative"
                :class="{
                  'border-pink-600': selectedLayout == 'A',
                  'border-gray-200': selectedLayout != 'A'
                }">
                <div class="relative z-20 w-15 h-48 p-0.5 flex flex-col gap-0.5 border rotate-5 bg-white">
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <p class="text-[8px]">withcu</p>
                </div>
                <div class="absolute ml-10 z-10 w-15 h-48 p-0.5 flex flex-col gap-0.5 border bg-white">
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <p class="text-[8px]">withcu</p>
                </div>
                <div class="w-10" />
              </button>
              <button
                @click="changeLayout('B')"
                class="border p-3 rounded flex justify-center items-center relative"
                :class="{
                  'border-pink-600': selectedLayout == 'B',
                  'border-gray-200': selectedLayout != 'B'
                }">
                <div class="relative z-10 w-15 h-48 p-0.5 flex flex-col gap-0.5 border rotate-2 bg-white">
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-12 flex flex-col justify-center items-center">
                    <p class="text-[8px]">withcu</p>
                    <p class="text-[8px]">2025.12.01</p>
                  </div>
                </div>
                <div class="absolute ml-10 z-20 w-15 h-48 p-0.5 flex flex-col gap-0.5 border bg-white">
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-12 flex flex-col justify-center items-center">
                    <p class="text-[8px]">withcu</p>
                    <p class="text-[8px]">2025.12.01</p>
                  </div>
                </div>
                <div class="w-10" />
              </button>
              <button
                @click="changeLayout('C')"
                class="border p-3 rounded flex justify-center items-center relative"
                :class="{
                  'border-pink-600': selectedLayout == 'C',
                  'border-gray-200': selectedLayout != 'C'
                }">
                <div class="relative z-10 w-15 h-48 p-0.5 flex flex-col gap-0.5 border rotate-5 bg-white">
                  <div class="w-full h-6 flex flex-col justify-center items-center">
                    <p class="text-[8px]">witchu</p>
                  </div>
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-6 flex flex-col justify-center items-center">
                    <p class="text-[8px]">2025.12.01</p>
                  </div>
                </div>
                <div class="absolute ml-10 z-20 w-15 h-48 p-0.5 flex flex-col gap-0.5 border bg-white">
                  
                  <div class="w-full h-6 flex flex-col justify-center items-center">
                    <p class="text-[8px]">witchu</p>
                  </div>
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-10 border" />
                  <div class="w-full h-6 flex flex-col justify-center items-center">
                    <p class="text-[8px]">2025.12.01</p>
                  </div>
                </div>
                <div class="w-10" />
              </button>
            </div>
          </div>
        </div>
      </div>
  </div>
</template>

<script setup lang="ts">
type Layouts = {
  header: boolean,
  footer: boolean,
  strip: number,
  vertical: boolean
}
type Props = {
  layout: Layouts
}

const props = defineProps<Props>()
const currentLayout = ref<Layouts>(props.layout)
const open = ref<boolean>(false)
const selectedLayout = ref<string>('C')

const emits = defineEmits<{
  (e: 'update:layout', value: Layouts): void
}>()

const changeModalState = () => {
  const newValue = !open.value
  open.value = newValue
}

const changeLayout = (type: string) => {
  switch (type) {
    case 'A':
      currentLayout.value = {
        header: false,
        footer: true,
        strip: 4,
        vertical: true
      }
      break;
    case 'B':
      currentLayout.value = {
        header: false,
        footer: true,
        strip: 3,
        vertical: true
      }
      break;
    default:
      currentLayout.value = {
        header: true,
        footer: true,
        strip: 3,
        vertical: true
      }
      break;
  }
  selectedLayout.value = type
  emits('update:layout', currentLayout.value)
  open.value = false
}
</script>