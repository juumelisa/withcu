<template>
  <div class="w-full min-h-full overflow-y-auto p-5 xl:p-0">
    <div class="flex w-full">
      <div class="w-full max-w-7xl mx-auto flex justify-between py-3 font-semibold text-blue-900 text-xl">
        <!-- <img src="~/assets/images/logo.png" class="h-12" /> -->
         withcu
      </div>
    </div>
    <div class="w-full">
      <div class="w-full max-w-7xl mx-auto">
        <div class="w-full flex flex-col-reverse md:flex-row justify-center md:justify-between items-center min-h-screen gap-10">
          <div>
            <h1 class="text-2xl md:text-3xl lg:text-5xl gap-1 mb-5 max-w-lg">
              Your personal <span class="text-pink-600">photobooth</span> studio!
            </h1>
            <p class="text-base md:text-lg max-w-lg">
              Capture, edit, and share amazing photos with fun filters and creative frames.
            </p>
            <div class="w-60 mt-5">
              <nuxt-link
                href="/frame-selection"
                class="py-3 px-5 flex items-center justify-center gap-2 bg-blue-900 text-white outline-0 rounded-full">
                Get Started
              </nuxt-link>
            </div>
          </div>
          <div class="flex relative">
            <div class="w-80 -rotate-6">
              <img src="~/assets/images/landing-image2.png" class="" />
            </div>
            <div class="w-40 rotate-1">
              <img src="~/assets/images/landing-image.png" class="" />
            </div>
            <div class="w-80 absolute left-0 bottom-10">
              <img src="~/assets/images/landing-image3.png" class="" />
            </div>
          </div>
        </div>
        <div class="mt-10 w-full grid grid-cols-1 md:grid-cols-3 gap-5">
          <div class="bg-gray-100/10 shadow shadow-white border border-white rounded-2xl p-5 flex flex-col justify-center items-center gap-2">
            <div class="shadow-white border border-white rounded-2xl p-3 bg-gray-400/20">
              <icons-camera class="size-5 text-white"/>
            </div>
            <h2 class="text-lg text-center text-blue-900 font-semibold">Live Camera</h2>
            <p class="text-center">
              Access your webcam instantly with real-time preview and countdown timer for perfect shots.
            </p>
          </div>
          <div class="bg-gray-100/10 shadow shadow-white border border-white rounded-2xl p-5 flex flex-col justify-center items-center gap-2">
            <div class="shadow-white border border-white rounded-2xl p-3 bg-gray-400/20">
              <icons-filter-pretty class="size-5 text-white"/>
            </div>
            <h2 class="text-lg text-center text-blue-900 font-semibold">Various Filter</h2>
            <p class="text-center">
              Transform your photos with vintage, cool, warm, noir, fade, and many more filters.
            </p>
          </div>
          <div class="bg-gray-100/10 shadow shadow-white border border-white rounded-2xl p-5 flex flex-col justify-center items-center gap-2">
            <div class="shadow-white border border-white rounded-2xl p-3 bg-gray-400/20">
              <icons-frame class="size-5 text-white"/>
            </div>
            <h2 class="text-lg text-center text-blue-900 font-semibold">Creative Frame</h2>
            <p class="text-center">
              Customize your frames to make your photos stand out.
            </p>
          </div>
          <div class="bg-gray-100/10 shadow shadow-white border border-white rounded-2xl p-5 flex flex-col justify-center items-center gap-2">
            <div class="shadow-white border border-white rounded-2xl p-3 bg-gray-400/20">
              <icons-camera class="size-5 text-white"/>
            </div>
            <h2 class="text-lg text-center text-blue-900 font-semibold">Live Camera</h2>
            <p class="text-center">
              Access your webcam instantly with real-time preview and countdown timer for perfect shots.
            </p>
          </div>
          <div class="bg-gray-100/10 shadow shadow-white border border-white rounded-2xl p-5 flex flex-col justify-center items-center gap-2">
            <div class="shadow-white border border-white rounded-2xl p-3 bg-gray-400/20">
              <icons-camera class="size-5 text-white"/>
            </div>
            <h2 class="text-lg text-center text-blue-900 font-semibold">Live Camera</h2>
            <p class="text-center">
              Access your webcam instantly with real-time preview and countdown timer for perfect shots.
            </p>
          </div>
          <div class="bg-gray-100/10 shadow shadow-white border border-white rounded-2xl p-5 flex flex-col justify-center items-center gap-2">
            <div class="shadow-white border border-white rounded-2xl p-3 bg-gray-400/20">
              <icons-camera class="size-5 text-white"/>
            </div>
            <h2 class="text-lg text-center text-blue-900 font-semibold">Live Camera</h2>
            <p class="text-center">
              Access your webcam instantly with real-time preview and countdown timer for perfect shots.
            </p>
          </div>
        </div>
        <div class="mt-50 w-full bg-white rounded-2xl p-5 md:p-10 mb-20">
          <h2 class="capitalize text-lg text-center">How it works</h2>
          <div class="w-full grid grid-cols-1 md:grid-cols-3 gap-5 md:gap-10 mt-5 md:mt-10">
            <div
              v-for="(instruction, index) in instructions"
              :key="instruction.label"
              class="flex flex-col justify-center items-center gap-3">
              <div class="w-16 h-16 bg-pink-100 flex justify-center items-center text-xl rounded-full">{{ index + 1 }}</div>
              <p class="text-lg text-pink-600">{{ instruction.label }}</p>
              <p class="text-center">{{ instruction.description }}</p>
            </div>
          </div>
        </div>
        <div class="my-50 text-lg text-blue-900 flex flex-col justify-center items-center gap-3">
          <p class="text-center">Ready to Create Amazing Photos?</p>
          <p class="text-center">No sign up required. No downloads. Just pure creative fun in your browser.</p>
          <div class="w-60">
            <nuxt-link
              href="/photo"
              class="py-3 px-5 flex items-center justify-center gap-2 bg-blue-900 text-white outline-0 rounded-full">
              Get Started
            </nuxt-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const instructions = [
  {
    label: "Capture",
    description: "Choose a filter and click capture. A 3-second countdown gives you time to pose!"
  },
  {
    label: "Edit",
    description: "Apply filters, frames, adjust colors, rotate, or draw on your photos in the editor."
  },
  {
    label: "Download",
    description: "Save your masterpiece and share it with friends or on social media!"
  }
]
</script>