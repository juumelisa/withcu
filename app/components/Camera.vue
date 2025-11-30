
<template>
  <div class="w-full flex justify-center">
    <video ref="videoRef" autoplay playsinline class="max-w-full rounded-lg"></video>
    <canvas ref="canvasRef" class="hidden"></canvas>
    <button @click="capture">Take Photo</button>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const videoRef = ref<HTMLVideoElement | null>(null)
const canvasRef = ref<HTMLCanvasElement | null>(null)
let stream: MediaStream | null = null

onMounted(async () => {
  try {
    stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'user' } // front camera
    })

    if (videoRef.value) {
      videoRef.value.srcObject = stream
      await videoRef.value.play()
    }
  } catch (error) {
    console.error('Camera error:', error)
  }
})

onBeforeUnmount(() => {
  if (stream) {
    stream.getTracks().forEach(track => track.stop())
  }
})

const capture = () => {
  if (videoRef.value && canvasRef.value) {
    const canvas = canvasRef.value
    canvas.width = videoRef.value.videoWidth
    canvas.height = videoRef.value.videoHeight

    const ctx = canvas.getContext('2d')
    if (ctx) {
        ctx.drawImage(videoRef.value, 0, 0, canvas.width, canvas.height)
        const dataUrl = canvas.toDataURL('image/png')
        console.log('captured image: ', dataUrl)
    }
  }
}
</script>


<style scoped>
video {
  transform: scaleX(-1); /* mirror camera like photobooth */
}
</style>
