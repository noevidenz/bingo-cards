<script setup>
import { sampleSize } from 'lodash'
import { computed, onMounted, ref } from 'vue'

import economy from './lists/economy.json'
import entertainment from './lists/entertainment.json'
import likely from './lists/likely.json'
import politics from './lists/politics.json'
import science from './lists/science.json'
import unlikely from './lists/unlikely.json'
import unpleasant from './lists/unpleasant.json'
import html2canvas from 'html2canvas'

const DEFAULT_GRID_OPTIONS = [
  ...economy,
  ...entertainment,
  ...likely,
  ...politics,
  ...science,
  ...unlikely,
  ...unpleasant,
]
const FREE_SQUARE = 'Free!'

const customOptionsRaw = ref('')
const customOptions = ref(false)
const includeDefaultOptions = ref(true)

const gridElement = ref(null)
const grid = ref([])

const hiddenDownloadLink = ref(null)

onMounted(() => {
  grid.value = Array(25)
  grid.value[12] = FREE_SQUARE
})

const customOptionsArray = computed(() => customOptionsRaw.value.trim().split('\n').filter(e => e))

const gridOptions = computed(() => {
  const options = []

  if (includeDefaultOptions.value) {
    options.push(...DEFAULT_GRID_OPTIONS)
  }

  if (customOptions.value) {
    options.push(...customOptionsArray.value)
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
</script>

<template>
  <div class="flex flex-col gap-y-8 items-center justify-center mt-16">
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
          :disabled="customOptions && !includeDefaultOptions && customOptionsArray.length < 24"
          class="bg-slate-100 border border-slate-600 px-2.5 py-2 rounded-lg text-slate-800 hover:bg-slate-200 disabled:bg-slate-100 disabled:opacity-80"
          @click="generate"
      >Generate!</button>

      <button
          class="bg-slate-100 border border-slate-600 px-2.5 py-2 rounded-lg text-slate-800 hover:bg-slate-200"
          @click="download"
      >Download</button>
    </div>

    <button class="underline" @click="customOptions = !customOptions">Use custom options</button>

    <div v-if="customOptions">

      <div class="my-4">
        <label>
          <input type="checkbox" v-model="includeDefaultOptions" class="mr-2.5">
          <span>Include default options?</span>
        </label>
      </div>

      <div class="my-4">
        <p>
          Enter custom options below, one item per line.
          <br>
          <span v-if="customOptionsArray.length < 24" class="text-red-600">Must supply at least 24 options! (<span v-text="24 - customOptionsArray.length"></span> more)</span>
        </p>
        <textarea v-model="customOptionsRaw" class="border border-slate-300 h-96 m-4 p-4 w-[800px]"></textarea>
      </div>
    </div>

    <a ref="hiddenDownloadLink" class="hidden" download="2023-bingo.png"></a>
  </div>
</template>

<style scoped>

</style>
