<template>
<div>
    <div v-bind="getRootProps()">
      <input v-bind="getInputProps()" />
      <p v-if="isDragActive">Drop the files here ...</p>
      <p v-else>Drag 'n' drop some files here, or click to select files</p>
    </div>
    <div class="vizualizer-screen-container">
        <VizualizerScreen />
    </div>
    <audio controls id="main-playback"></audio>
</div>
</template>

<script lang="ts" setup>
import { useDropzone } from "vue3-dropzone"
import { ref, onMounted } from 'vue'
import VizualizerScreen from "./VizualizerScreen.vue"

let isPlaying = ref(false)

// dropzone
function onDrop(acceptFiles, rejectReasons) {
  console.log(acceptFiles)
  console.log(rejectReasons)
  const file = acceptFiles[0]
  console.log(file)
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
  audioCtx.decodeAudioData(arrayBuffer).then((buffer) => {
    console.log(buffer)
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
    isPlaying.value = true
    draw()
  })
  audio.addEventListener('pause', () => {
    isPlaying.value = false
    cancelAnimationFrame(draw)
  })
})

// audio analyser
const audioCtx = new (window.AudioContext || window.webkitAudioContext)()
const analyser = audioCtx.createAnalyser()
analyser.fftSize = 2048
const bufferLength = analyser.frequencyBinCount
const dataArray = new Uint8Array(bufferLength)
analyser.getByteTimeDomainData(dataArray)

function draw() {
  if (!isPlaying.value) {
    return
  }
  requestAnimationFrame(draw)
  analyser.getByteTimeDomainData(dataArray)
  console.log(dataArray)
}

</script>

<style scoped>
.vizualizer-screen-container {
  width: 100%;
  height: 90vh;
}
</style>