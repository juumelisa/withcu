<template>
  <div>
    <div
      class="w-full max-h-full relative z-10 bg-white">
      <h1 class="p-5">Change Layout</h1>
      <div class="h-full overflow-y-auto p-5">
        <div class="w-full grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-5">
          <button
            @click="changeLayout('A')"
            class="border border-gray-200 p-3 rounded flex justify-center items-center relative"
            :class="{
              'border-blue-50 bg-blue-50': selectedLayout == 'A',
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
              'border-blue-50 bg-blue-50': selectedLayout == 'B',
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
              'border-blue-50 bg-blue-50': selectedLayout == 'C',
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
const selectedLayout = ref<string>('C')

const emits = defineEmits<{
  (e: 'update:layout', value: Layouts): void
}>()

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
}
</script>