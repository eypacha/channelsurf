<template>
  <canvas v-show="show" ref="canvasRef" class="fixed inset-0 w-full h-full z-30 pointer-events-none" />
</template>

<script setup>
import { ref, watch, onMounted, onBeforeUnmount, defineProps, defineExpose } from 'vue'

const props = defineProps({
  show: Boolean,
  isMuted: Boolean // <-- Añadido para recibir el estado de mute
})

const canvasRef = ref(null)
let animationFrameId = null
let audioCtx = null
let noiseSource = null

function drawWhiteNoise() {
  if (!props.show) return
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const width = window.innerWidth / 2
  const height = window.innerHeight / 2
  if (canvas.width !== width || canvas.height !== height) {
    canvas.width = width
    canvas.height = height
  }
  const imageData = ctx.createImageData(width, height)
  const buffer = new Uint32Array(imageData.data.buffer)
  for (let i = 0; i < buffer.length; i++) {
    const color = Math.random() > 0.5 ? 0xffffffff : 0xff000000
    buffer[i] = color
  }
  ctx.putImageData(imageData, 0, 0)
  animationFrameId = requestAnimationFrame(drawWhiteNoise)
}

function stopWhiteNoise() {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
    animationFrameId = null
  }
}

function startWhiteNoiseAudio() {
  if (props.isMuted) return // <-- No iniciar audio si está muteado
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)()
  }
  if (noiseSource) return
  const bufferSize = 2 * audioCtx.sampleRate
  const noiseBuffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate)
  const output = noiseBuffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    output[i] = Math.random() * 2 - 1
  }
  noiseSource = audioCtx.createBufferSource()
  noiseSource.buffer = noiseBuffer
  noiseSource.loop = true
  noiseSource.connect(audioCtx.destination)
  noiseSource.start(0)
}

function stopWhiteNoiseAudio() {
  if (noiseSource) {
    noiseSource.stop()
    noiseSource.disconnect()
    noiseSource = null
  }
}

defineExpose({ drawWhiteNoise, stopWhiteNoise, startWhiteNoiseAudio, stopWhiteNoiseAudio, canvasRef })

watch(() => props.show, (val) => {
  if (val) {
    drawWhiteNoise()
  } else {
    stopWhiteNoise()
  }
})

// Nuevo watcher para isMuted
watch(() => props.isMuted, (val) => {
  if (val) {
    stopWhiteNoiseAudio()
  } else if (props.show) {
    startWhiteNoiseAudio()
  }
})

onMounted(() => {
  if (props.show) drawWhiteNoise()
})

onBeforeUnmount(() => {
  stopWhiteNoise()
  stopWhiteNoiseAudio()
})
</script>

<style scoped>
canvas {
  display: block;
}
</style>
