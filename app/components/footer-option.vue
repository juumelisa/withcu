<template>
  <div>
    <button @click="changeModalState" class="border border-gray-200">
      <icons-footer class="size-10" />
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
          class="w-auto absolute z-10 right-16 top-20 bg-white rounded-xl shadow-xl transform transition-all duration-300 flex flex-col p-5"
          :class="{
            'scale-100 opacity-100': open,
            'scale-95 opacity-0': !open
          }"
        >
          <h1 class="text-lg">Change Background</h1>
          <!-- <div class="grid grid-cols-2">
            <button></button>
          </div> -->
          <div class="w-full flex flex-col justify-center items-center mt-5">
            <div class="w-30 h-90">
              <img v-if="backgroundImage" :src="backgroundImage" class="w-full" />
            </div>
            <div class="relative mt-5">
              <div class="p-3 border border-gray-200 rounded-full bg-linear-to-r from-pink-100 to-blue-300">
                <p class="text-blue-900 font-semibold">Click here to change background</p>
              </div>
              <input type="file" @change="onChangeFile" class="absolute top-0 left-0 h-full w-full opacity-0"/>
            </div>
          </div>
        </div>
      </div>
  </div>
</template>

<script setup lang="ts">
type Props = {
  backgroundImage?: string
}
const props = defineProps<Props>()
const open = ref<boolean>(false)

const emits = defineEmits<{
  (e: 'onChangeFooter', value: Event): void
}>()
const changeModalState = () => {
  const newValue = !open.value
  open.value = newValue
}

const onChangeFile = (value: Event) => {
  emits('onChangeFooter', value)
}
</script>