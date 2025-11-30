<template>
  <div>
    <button @click="changeModalState" class="border border-gray-200">
      <icons-collage class="size-10" />
    </button>
    <div
        class="fixed inset-0 z-50 flex items-center justify-center p-5 md:p-0 transition-all duration-300"
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
          class="w-full max-w-lg relative z-10 bg-white rounded-xl p-5 shadow-xl transform transition-all duration-300"
          :class="{
            'scale-100 opacity-100': open,
            'scale-95 opacity-0': !open
          }"
        >
          <h1>Change Layout</h1>
        </div>
      </div>
  </div>
</template>

<script setup lang="ts">
const open = ref<boolean>(false)

const changeModalState = () => {
  const newValue = !open.value
  open.value = newValue
}
</script>