<script setup lang="ts">
import { Ref, ref, watch } from 'vue'

const cameraRef: Ref<HTMLVideoElement> = ref<HTMLVideoElement>() as any
const canvasRef: Ref<HTMLCanvasElement> = ref<HTMLCanvasElement>() as any

function getImageFromCamera() {
  const canvas = canvasRef.value
  const context = canvas.getContext('2d')!
  const video = cameraRef.value

  context.drawImage(video, 0, 0, 100, 100)
  return canvas.toDataURL('image/png')
}

function loadImageToCanvas() {
  getImageFromCamera()

  requestAnimationFrame(loadImageToCanvas)
}

function openCamera() {
  navigator.mediaDevices
    .getUserMedia({ video: true, audio: false })
    .then((stream) => {
      if (cameraRef) {
        cameraRef.value.srcObject = stream
        cameraRef.value.onloadedmetadata = () => {
          cameraRef.value.width = cameraRef.value.videoWidth
          cameraRef.value.height = cameraRef.value.videoHeight

          requestAnimationFrame(loadImageToCanvas)
        }
      }
    })
    .catch((err) => {
      console.log(
        "May the browser didn't support or there is some errors.",
        err
      )
    })
}
</script>

<template>
  <div class="h-screen p-10">
    <div class="bg-purple-400">
      <video ref="cameraRef" :width="450" :height="337.5" autoplay></video>
    </div>
    <div class="bg-pink-400">
      <canvas ref="canvasRef"></canvas>
    </div>
    <div>
      <button @click="openCamera">Open Camera</button>
    </div>
  </div>
</template>
