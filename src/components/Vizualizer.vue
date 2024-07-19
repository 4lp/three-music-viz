<template>
  <div>
    <button>Use default music</button>
    <div v-bind="getRootProps()">
      <input v-bind="getInputProps()" />
      <p v-if="isDragActive">Drop the files here ...</p>
      <p v-else>Drag 'n' drop some files here, or click to select files</p>
    </div>
    <div class="vizualizer-screen-container">
      <VizualizerScreen :fftArray="dataArray" :byteFrequencyData="byteFrequencyData" />
    </div>
    <audio controls id="main-playback" :src="song"></audio>
  </div>
</template>

<script lang="ts" setup>
import { useDropzone } from 'vue3-dropzone'
import { ref, onMounted } from 'vue'
import VizualizerScreen from '@/components/VizualizerScreen.vue'
import { FFT_SIZE } from '@/utils/constants'
import song from '@/assets/song.mp3'

let analyser
let dataArray = new Uint8Array(FFT_SIZE)
let byteFrequencyData = new Uint8Array(FFT_SIZE)
let audioCtx

// dropzone
function onDrop(acceptFiles, rejectReasons) {
  const file = acceptFiles[0]
  // use as audio src
  const audio = document.getElementById('main-playback') as HTMLAudioElement
  audio.src = URL.createObjectURL(file)
  // read for vizualizer
  const reader = new FileReader()
  reader.readAsArrayBuffer(file)
  reader.onload = (evt) => {
    console.log(evt)
    onRead(evt.target.result)
  }
}
const { getRootProps, getInputProps } = useDropzone({ onDrop })
// convert audio file to array buffer
function onRead(arrayBuffer) {
  console.log(arrayBuffer)
  // audio analyser
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    analyser = audioCtx.createAnalyser()
    analyser.fftSize = FFT_SIZE
  }
  //analyser.getByteTimeDomainData(dataArray)
  audioCtx.decodeAudioData(arrayBuffer).then((buffer) => {
    const audio = document.getElementById('main-playback') as HTMLAudioElement
    const source = audioCtx.createMediaElementSource(audio)
    source.buffer = buffer
    source.connect(analyser)
    analyser.connect(audioCtx.destination)
  })
}
// audio playback

onMounted(() => {
  const audio = document.getElementById('main-playback') as HTMLAudioElement
  audio.addEventListener('play', () => {
    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)()
      analyser = audioCtx.createAnalyser()
      analyser.fftSize = FFT_SIZE
      const audio = document.getElementById('main-playback') as HTMLAudioElement
      const source = audioCtx.createMediaElementSource(audio)
      source.connect(analyser)
      analyser.connect(audioCtx.destination)
    }
    draw()
  })
})

function draw() {
  requestAnimationFrame(draw)
  analyser.getByteTimeDomainData(dataArray)
  analyser.getByteFrequencyData(byteFrequencyData)
}
</script>

<style scoped>
.vizualizer-screen-container {
  width: 100%;
  height: 90vh;
}
</style>
