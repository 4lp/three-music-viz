<template>
  <TresCanvas clear-color="#82DBC5">
    <TresPerspectiveCamera :position="[10, 10, 10]" :look-at="[0, 0, 0]" />
    <Scene :fftArray="props.fftArray" :byteFrequencyData="byteFrequencyData" />
    <TresAmbientLight :intensity="1" />
    <EffectComposer>
      <DepthOfField />
      <Bloom />
      <Pixelation />
    </EffectComposer>
  </TresCanvas>
</template>

<script lang="ts" setup>
import { TresCanvas } from '@tresjs/core'
import { EffectComposer, DepthOfField, Bloom, Pixelation } from '@tresjs/post-processing'
import Scene from './Scene.vue'

const props = defineProps({
  fftArray: {
    type: Uint8Array,
    //default: () => new Uint8Array(0),
    required: true
  },
  byteFrequencyData: {
    type: Uint8Array,
    //default: () => new Uint8Array(0),
    required: true
  }
})
</script>
