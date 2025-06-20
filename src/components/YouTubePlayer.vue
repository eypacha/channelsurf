<template>
  <div class="video-container bg-black h-[100dvh]">
    <div class="video-wrapper relative h-[100dvh] w-full overflow-hidden">
      <div id="player" class="absolute outline- h-[calc(100dvh+1000px)] w-full top-[-500px]"></div>
      <div class="absolute inset-0 cursor-pointer" @click="nextVideo"></div>
      <WhiteNoiseCanvas :show="showNoise" :isMuted="isMuted" ref="whiteNoiseRef" />
      <!-- Botón de ayuda -->
      <button
        class="absolute top-4 right-4 z-40 bg-white/80 hover:bg-white text-black w-8 h-8 flex items-center justify-center text-xl font-bold shadow cursor-pointer"
        @click="showHelp = true"
        :aria-label="$t('player.help')"
      >?</button>
      <!-- Modal de ayuda -->
      <div
        v-if="showHelp"
        class="fixed inset-0 z-50 flex items-center justify-center bg-black/60"
        @click.self="showHelp = false"
      >
        <div class="bg-black text-gray-100 shadow-lg p-6 max-w-xs w-full relative">
          <button
            class="absolute top-2 right-4 text-gray-100 hover:text-white text-xl cursor-pointer"
            @click="showHelp = false"
            :aria-label="$t('player.close')"
          >&times;</button>
          <h2 class="text-lg font-semibold mb-2">{{ $t('player.helpTitle') }}</h2>
          <p class="mb-4 text-sm">{{ $t('player.helpSubtitle') }}</p>
          <h3 class="font-semibold mb-2 text-sm">{{ $t('player.quickControls') }}</h3>
          <ul class="text-sm space-y-1">
            <li><kbd class="bg-gray-100 text-black px-1">F</kbd> — {{ $t('player.fullscreen') }}</li>
            <li><kbd class="bg-gray-100 text-black px-1">M</kbd> — {{ $t('player.mute') }}</li>
            <li>
              <kbd class="bg-gray-100 text-black px-1">{{ $t('player.space') }}</kbd>
              o
              <kbd class="bg-gray-100 text-black px-1">→</kbd>
              — {{ $t('player.next') }}
            </li>
            <li><kbd class="bg-gray-100 text-black px-1">←</kbd> — {{ $t('player.prev') }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onBeforeUnmount } from 'vue'
import WhiteNoiseCanvas from './WhiteNoiseCanvas.vue'

const props = defineProps({
  videoIds: {
    type: Array,
    required: true
  }
})

const currentIndex = ref(0)
let player = null
const showNoise = ref(true)
const whiteNoiseRef = ref(null)
const isMuted = ref(false)
const showHelp = ref(false)

function isValidVideoId(id) {
  return typeof id === 'string' && /^[a-zA-Z0-9_-]{11}$/.test(id)
}

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

function createPlayer() {
  const id = props.videoIds[currentIndex.value]
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
      onError: onPlayerError,
      onReady: (event) => {
        if (isMuted.value) {
          event.target.mute()
        } else {
          event.target.unMute()
        }
      }
    }
  })
}

function onPlayerStateChange(event) {
  if (event.data === window.YT.PlayerState.BUFFERING || event.data === window.YT.PlayerState.UNSTARTED || event.data === window.YT.PlayerState.CUED) {
    showNoise.value = true
    if (!isMuted.value) {
        whiteNoiseRef.value?.startWhiteNoiseAudio()
    }
  } else if (event.data === window.YT.PlayerState.PLAYING) {
    showNoise.value = false
    whiteNoiseRef.value?.stopWhiteNoiseAudio()
  } else if (event.data === window.YT.PlayerState.ENDED) {
    nextVideo()
  }
}

function onPlayerError(event) {
  const id = props.videoIds[currentIndex.value]
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
  nextVideo()
}

function nextVideo() {
  currentIndex.value = (currentIndex.value + 1) % props.videoIds.length
  const id = props.videoIds[currentIndex.value]
  if (!isValidVideoId(id)) {
    console.error(`ID de video no válido: ${id}`)
    return
  }
  if (player && player.loadVideoById) {
    showNoise.value = true
    whiteNoiseRef.value?.startWhiteNoiseAudio()
    player.loadVideoById(id)
    if (isMuted.value && player.mute) player.mute()
    if (!isMuted.value && player.unMute) player.unMute()
  }
}

function prevVideo() {
  currentIndex.value = (currentIndex.value - 1 + props.videoIds.length) % props.videoIds.length
  const id = props.videoIds[currentIndex.value]
  if (!isValidVideoId(id)) {
    console.error(`ID de video no válido: ${id}`)
    return
  }
  if (player && player.loadVideoById) {
    showNoise.value = true
     if (!isMuted.value) {
        whiteNoiseRef.value?.startWhiteNoiseAudio()
     }
    player.loadVideoById(id)
    if (isMuted.value && player.mute) player.mute()
    if (!isMuted.value && player.unMute) player.unMute()
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

function handleKeydown(e) {
    switch (true) {
        case e.code === 'Space' || e.code === 'ArrowRight':
            e.preventDefault()
            nextVideo()
            break
        case e.code === 'ArrowLeft':
            e.preventDefault()
            prevVideo()
            break
        case e.key === 'm' || e.key === 'M':
            isMuted.value = !isMuted.value
            if (player && player.mute && player.unMute) {
                if (isMuted.value) {
                    player.mute()
                } else {
                    player.unMute()
                }
            }
            break
        case e.key === 'f' || e.key === 'F':
            toggleFullscreen()
            break
    }
}

function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
}

onMounted(async () => {
  shuffleArray(props.videoIds)
  await loadYouTubeAPI()
  createPlayer()
  showNoise.value = true
  whiteNoiseRef.value?.startWhiteNoiseAudio()
  window.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
#player iframe {
  border: none;
}
</style>
