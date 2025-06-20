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
  'CtmxTj9pKqg',
  '5DaVnriHhPc',
  'V_5osvOhtZ4',
  '9hiCRevJlv0',
  'AI9Yroa_Gz8',
  'YPW_HeOHA1E',
  'srBHgLGbG8I',
  'tCRPUv8V22o',
  'RYJMZ8FAwLE'
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