<template>
  <div class="home-view">
    <div class="video-container bg-black h-[100dvh]">
      <div class="video-wrapper relative h-[100dvh] w-full overflow-hidden">
        <button class="absolute top-4 right-4 z-20 bg-black/60 rounded-full cursor-pointer p-2 hover:bg-black/80 transition" @click="toggleFullscreen">
          <img src="@/assets/fullscreen.svg" alt="Pantalla completa" class="w-7 h-7" />
        </button>
        <div id="player" class="absolute outline- h-[calc(100dvh+1000px)] w-full top-[-500px]"></div>
        <div class="absolute inset-0 cursor-pointer" @click="nextVideo"></div>
        <canvas v-show="showNoise" ref="noiseCanvas" class="fixed inset-0 w-full h-full z-30 pointer-events-none" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const videoIds = [
  'k3GKGDng7k0',
  'xdS3-eTuRjM',
  'yHJOz_y9rZE',
  'I9_3CfRm8GE',
  'qnL40CbuodU',
  'qP1hJLepOhw',
  'th1bLBuLdc4',
  'ij8M2RZTPCo',
  'CtmxTj9pKqg'

]
const currentIndex = ref(0)
let player = null
const noiseCanvas = ref(null)
let animationFrameId = null
const showNoise = ref(true)
let audioCtx = null
let noiseSource = null

function loadYouTubeAPI() {
  return new Promise((resolve) => {
    if (window.YT && window.YT.Player) {
      resolve(window.YT)
    } else {
      const tag = document.createElement('script')
      tag.src = 'https://www.youtube.com/iframe_api'
      window.onYouTubeIframeAPIReady = () => resolve(window.YT)
      document.body.appendChild(tag)
    }
  })
}

function isValidVideoId(id) {
  // YouTube video IDs are 11 characters, alphanumeric, - and _
  return typeof id === 'string' && /^[a-zA-Z0-9_-]{11}$/.test(id)
}

function createPlayer() {
  const id = videoIds[currentIndex.value]
  if (!isValidVideoId(id)) {
    console.error(`ID de video no válido: ${id}`)
    return
  }
  player = new window.YT.Player('player', {
    height: '390',
    width: '640',
    videoId: id,
    playerVars: {
      autoplay: 1,
      controls: 0,
      modestbranding: 1,
      rel: 0,
      showinfo: 0,
      fs: 0,
      disablekb: 1,
      iv_load_policy: 3,
      playsinline: 1
    },
    events: {
      onStateChange: onPlayerStateChange,
      onError: onPlayerError
    }
  })
}

function startWhiteNoiseAudio() {
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

function onPlayerStateChange(event) {
  // 3 = buffering, 2 = paused, 1 = playing, 0 = ended, 5 = cued, -1 = unstarted
  if (event.data === window.YT.PlayerState.BUFFERING || event.data === window.YT.PlayerState.UNSTARTED || event.data === window.YT.PlayerState.CUED) {
    showNoise.value = true
    drawWhiteNoise()
    startWhiteNoiseAudio()
  } else if (event.data === window.YT.PlayerState.PLAYING) {
    showNoise.value = false
    cancelAnimationFrame(animationFrameId)
    stopWhiteNoiseAudio()
  } else if (event.data === window.YT.PlayerState.ENDED) {
    nextVideo()
  }
}

function onPlayerError(event) {
  const id = videoIds[currentIndex.value]
  let message = ''
  switch (event.data) {
    case 2:
      message = `ID de video no válido: ${id}`
      break
    case 5:
      message = `El reproductor no puede reproducir este video HTML5: ${id}`
      break
    case 100:
      message = `El video no existe o es privado: ${id}`
      break
    case 101:
    case 150:
      message = `El propietario no permite la reproducción en este sitio: ${id}`
      break
    default:
      message = `Error desconocido con el video: ${id}`
  }
  console.error(message)
  // Pasar al siguiente video automáticamente
  nextVideo()
}

function nextVideo() {
  currentIndex.value = (currentIndex.value + 1) % videoIds.length
  const id = videoIds[currentIndex.value]
  if (!isValidVideoId(id)) {
    console.error(`ID de video no válido: ${id}`)
    return
  }
  if (player && player.loadVideoById) {
    showNoise.value = true
    drawWhiteNoise()
    startWhiteNoiseAudio()
    player.loadVideoById(id)
  }
}

function toggleFullscreen() {
  const el = document.querySelector('.video-container')
  if (!el) return
  if (document.fullscreenElement) {
    document.exitFullscreen()
  } else {
    el.requestFullscreen()
  }
}

function drawWhiteNoise() {
  if (!showNoise.value) return
  const canvas = noiseCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const width = window.innerWidth
  const height = window.innerHeight
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

function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
}

onMounted(async () => {
  shuffleArray(videoIds)
  await loadYouTubeAPI()
  createPlayer()
  showNoise.value = true
  drawWhiteNoise()
  startWhiteNoiseAudio()
})
</script>

<style scoped>
#player iframe {
  border: none;
}
canvas {
  display: block;
}
</style>