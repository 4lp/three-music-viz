<template>
  <primitive :object="mesh" />
</template>

<script lang="ts" setup>
import { TresCanvas } from '@tresjs/core'
import { FFT_SIZE, colors, params, vertex, fragment } from '@/utils/constants'
import { useLoop } from '@tresjs/core'
import { IcosahedronGeometry, ShaderMaterial, Vector2, Mesh } from 'three'
import { GUI } from 'dat.gui'

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

const gui = new GUI()
const colorsFolder = gui.addFolder('Colors')
colorsFolder.add(params, 'red', 0, 1).onChange((value) => {
  uniforms.u_red.value = Number(value)
})
colorsFolder.add(params, 'blue', 0, 1).onChange((value) => {
  uniforms.u_blue.value = Number(value)
})
colorsFolder.add(params, 'green', 0, 1).onChange((value) => {
  uniforms.u_green.value = Number(value)
})

const uniforms = {
  u_time: { type: 'f', value: 0.0 },
  u_frequency: { type: 'f', value: 0.0 },
  u_red: { type: 'f', value: 1.0 },
  u_green: { type: 'f', value: 1.0 },
  u_blue: { type: 'f', value: 1.0 }
}

const material = new ShaderMaterial({
  uniforms,
  vertexShader: vertex,
  fragmentShader: fragment
})
const geometry = new IcosahedronGeometry(4, 30)
const mesh = new Mesh(geometry, material)
mesh.material.wireframe = true

const { onBeforeRender } = useLoop()

onBeforeRender(({ delta, elapsed }) => {
  uniforms.u_time.value = elapsed
  const average =
    props.byteFrequencyData.reduce((acc, val) => acc + val, 0) / props.byteFrequencyData.length
  uniforms.u_frequency.value = average
  mesh.rotation.x = elapsed
  console.log(props.byteFrequencyData)
})
</script>
