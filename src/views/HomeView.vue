<template>
  <div class="home-view">
    <YouTubePlayer :video-ids="videoIds" />
  </div>
</template>

<script setup>
import YouTubePlayer from '@/components/YouTubePlayer.vue'

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
</script>

<style scoped>
#player iframe {
  border: none;
}
canvas {
  display: block;
}
</style>