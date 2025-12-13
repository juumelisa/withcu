<template>
  <div>
    <label v-if="label" class="capitalize block mb-1">{{ label }}</label>
    <input
      :type="type"
      :placeholder="placeholder"
      class="w-full bg-white rounded-full border border-pink-400 px-5 py-2.5 outline-0" />
  </div>
</template>

<script setup lang="ts">
  type Props = {
    label?: string,
    name?: string,
    type: string | 'text',
    placeholder?: string
  }

  const props = defineProps<Props>()
</script>