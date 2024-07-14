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
import VizualizerScreen from "./VizualizerScreen.vue"

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
// convert audio file to array buffer
function onRead(arrayBuffer) {
  console.log(arrayBuffer)
  audioCtx.decodeAudioData(arrayBuffer).then((buffer) => {
    console.log(buffer)
    const source = audioCtx.createBufferSource()
    source.buffer = buffer
    source.connect(analyser)
    analyser.connect(audioCtx.destination)
    //source.start()
  })
}
const { getRootProps, getInputProps } = useDropzone({ onDrop })
// audio analyser
const audioCtx = new (window.AudioContext || window.webkitAudioContext)()

const analyser = audioCtx.createAnalyser()
analyser.fftSize = 2048

const bufferLength = analyser.frequencyBinCount
const dataArray = new Uint8Array(bufferLength)
analyser.getByteTimeDomainData(dataArray)

</script>

<style scoped>
.vizualizer-screen-container {
  width: 100%;
  height: 90vh;
}
</style>