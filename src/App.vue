<script setup>
import html2canvas from 'html2canvas'
import { sampleSize } from 'lodash'
import { computed, onMounted, reactive, ref } from 'vue'

import DEFAULT_GRID_OPTIONS from './lists/index.js'

const FREE_SQUARE = 'Free!'

const customOptionsForm = reactive({
  enabled: false,
  includeDefault: true,
  input: localStorage.getItem('customOptions') || '',
})

const grid = ref([])

const gridElement = ref(null)
const hiddenDownloadLink = ref(null)

onMounted(() => {
  grid.value = Array(25)
  grid.value[12] = FREE_SQUARE
})

const customOptions = computed(() => customOptionsForm.input.trim().split('\n').filter(e => e))

const gridOptions = computed(() => {
  const options = []

  if (customOptionsForm.includeDefault) {
    options.push(...DEFAULT_GRID_OPTIONS)
  }

  if (customOptionsForm.enabled) {
    options.push(...customOptions.value)
  }

  return options
})

const download = () => {
  html2canvas(gridElement.value).then((canvas) => {
    canvas.toBlob((blob) => {
      hiddenDownloadLink.value.href = URL.createObjectURL(blob)

      hiddenDownloadLink.value.click()
    })
  })
}

const generate = () => {
  // Randomly select 24 options from the list, then insert the
  // "Free!" square in the middle of the array to make up 25
  const subset = sampleSize(gridOptions.value, 24)

  subset.splice(12, 0, FREE_SQUARE)

  grid.value = subset
}

const storeCustomOptions = () => localStorage.setItem('customOptions', customOptionsForm.input)
</script>

<template>
  <a
      class="absolute right-6 top-4 block opacity-50 w-10 hover:opacity-100"
      href="https://github.com/noevidenz/bingo-cards"
      title="Github Repo"
      target="_blank"
  >
    <img src="/src/assets/github-mark.svg" class="aspect-square" alt="Github">
  </a>

  <div class="flex flex-col gap-y-8 items-center justify-center pt-16">
    <h1 class="font-bold text-4xl text-slate-700">2023 Bingo!</h1>

    <div ref="gridElement" class="bg-black border border-black grid gap-px bingo-card text-black">
      <div
          v-for="(square, index) in grid" :key="index"
          v-text="square"
          class="bg-white flex items-center justify-center p-4 text-center text-sm hover:bg-gray-50"
      ></div>
    </div>

    <div class="flex gap-4">
      <button
          :disabled="customOptionsForm.enabled && !customOptionsForm.includeDefault && customOptions.length < 24"
          class="bg-slate-100 border border-slate-600 px-2.5 py-2 rounded-lg text-slate-800 hover:bg-slate-200 disabled:bg-slate-100 disabled:opacity-80"
          @click="generate"
      >Generate!</button>

      <button
          class="bg-slate-100 border border-slate-600 px-2.5 py-2 rounded-lg text-slate-800 hover:bg-slate-200"
          @click="download"
      >Download</button>
    </div>

    <button class="underline" @click="customOptionsForm.enabled = !customOptionsForm.enabled">Use custom options</button>

    <div v-if="customOptionsForm.enabled" class="p-8 mx-auto max-w-4xl w-full">

      <div class="my-4">
        <label>
          <input type="checkbox" v-model="customOptionsForm.includeDefault" class="mr-2.5">
          <span>Include default options?</span>
        </label>
      </div>

      <div class="my-4">
        <p class="my-4">
          Enter custom options below, one item per line.
          <br>
          <span v-if="!customOptionsForm.includeDefault && customOptions.length < 24" class="text-red-600">Must supply at least 24 options! (<span v-text="24 - customOptions.length"></span> more)</span>
        </p>
        <textarea @blur="storeCustomOptions" v-model="customOptionsForm.input" class="border border-slate-300 h-96 p-4 w-full"></textarea>
      </div>
    </div>

    <a ref="hiddenDownloadLink" class="hidden" download="2023-bingo.png"></a>
  </div>

</template>

<style scoped>

</style>
