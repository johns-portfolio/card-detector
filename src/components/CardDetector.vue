<script setup lang="ts">
import { onMounted, Ref, ref, watch } from 'vue'

import '@tensorflow/tfjs-backend-cpu'
import '@tensorflow/tfjs-backend-webgl'
import * as cocoSsd from '@tensorflow-models/coco-ssd'

const cameraRef: Ref<HTMLVideoElement> = ref<HTMLVideoElement>() as any
const canvasRef: Ref<HTMLCanvasElement> = ref<HTMLCanvasElement>() as any
const canvasRef2: Ref<HTMLCanvasElement> = ref<HTMLCanvasElement>() as any

onMounted(async () => {
  const model = await cocoSsd.load()
  const video = cameraRef.value
  const canvas = canvasRef.value
  const ctx = canvas.getContext('2d')!
  const ctx2 = canvasRef2.value.getContext('2d')!

  const renderPrediction = async () => {
    const predictions = await model.detect(video)
    ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height)
    ctx.drawImage(video, 0, 0, ctx.canvas.width, ctx.canvas.height)

    // Draw the bounding box.
    predictions.forEach(prediction => {
      const x = prediction.bbox[0]
      const y = prediction.bbox[1]
      const width = prediction.bbox[2]
      const height = prediction.bbox[3]

      // Draw the bounding box.
      ctx.strokeStyle = '#00FFFF'
      ctx.lineWidth = 4
      ctx.strokeRect(x, y, width, height)

      // Draw the label 
      ctx.font = '18px Arial'
      ctx.fillStyle = '#00FFFF'
      ctx.fillText(prediction.class, x, y)

    })

    const bookPrediction = predictions.filter((c) => c.class === 'book')[0]
    if (bookPrediction) {
      const [x, y, width, height] = bookPrediction.bbox

      const croppedImage = ctx.getImageData(x, y, width, height)
      ctx2.putImageData(croppedImage, 0, 0)
    }
    requestAnimationFrame(renderPrediction)
  }

  const startVideo = async () => {
    const stream = await navigator.mediaDevices.getUserMedia({
      audio: false,
      video: {
        facingMode: 'environment'
      }
    })
    video.srcObject = stream
    video.onloadedmetadata = async () => {
      cameraRef.value.width = cameraRef.value.videoWidth
      cameraRef.value.height = cameraRef.value.videoHeight
      canvasRef.value.width = cameraRef.value.videoWidth
      canvasRef.value.height = cameraRef.value.videoHeight
      video.play()
      await renderPrediction()
    }
  }

  await startVideo()
})
</script>

<template>
  <div class="h-screen p-10 grid grid-cols-2">
    <div class="bg-purple-400">
      <video ref="cameraRef" autoplay></video>
    </div>
    <div class="bg-pink-400">
      <canvas ref="canvasRef"></canvas>
    </div>
    <div class="bg-pink-400">
      <canvas ref="canvasRef2"></canvas>
    </div>
    <div>
      <!-- <button @click="openCamera">Open Camera</button> -->
    </div>
  </div>
</template>
