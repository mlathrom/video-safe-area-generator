<template>
  <div class="max-w-screen-md mx-auto px-6">
    <div>
      <div class="text-center py-8">
        <h1 class="text-2xl font-bold">Safe Guides Generator</h1>
        <p class="mt-2">The resolution will comform to the selected aspect ratio.</p>
        <p class="mt-2">Select "Custom" to enter a custom resolution.</p>
      </div>
      <div class="">
        <form @submit.prevent="generateGuides">
          <div class="">
            <div class="flex max-w-lg lg:text-lg mx-auto space-x-2">
              <div class="w-1/3 bg-slate-700 rounded-lg">
                <label for="aspect-ratio" class="block px-3 py-1 shadow-md text-sm font-bold text-slate-300">Aspect</label>
                <select v-model="selectedAspectRatio" class="form-select bg-transparent rounded border-none w-full lg:text-lg custom-select outline-none focus:outline-none">
                  <option value="custom">Custom</option>
                  <option value="4:3">4:3</option>
                  <option value="16:9">16:9</option>
                  <option value="1.85:1">1.85:1</option>
                  <option value="2.39:1">2.39:1</option>
                  <option value="2.76:1">2.76:1</option>
                  <option value="1.37:1">1.37:1</option>
                  <option value="2.59:1">2.59:1</option>
                  <option value="2.65:1">2.65:1</option>
                  <option value="2.35:1">2.35:1</option>
                  <option value="2.66:1">2.66:1</option>
                </select>
              </div>
              <div class="w-1/3 bg-slate-700 rounded-lg">
                <label for="width" class="block px-3 py-1 shadow-md text-sm font-bold text-slate-300">Width</label>
                <input type="number" v-model.number="userInputWidth" min="1" class="form-input bg-transparent border-none w-full lg:text-lg outline-none focus:outline-none">
              </div>
              <div class="w-1/3 bg-slate-700 rounded-lg">
                <label for="height" class="block px-3 py-1 shadow-md text-sm font-bold text-slate-300">Height</label>
                <input type="number" v-model.number="userInputHeight" min="1" class="form-input bg-transparent border-none w-full lg:text-lg outline-none focus:outline-none">
              </div>
            </div>
            <div class="mx-auto justify-center max-w-lg mt-4">
              <div>
                <button type="submit" class="px-4 py-2 bg-teal-700 rounded-lg w-full">Generate Guides</button>
              </div>
              <div>
              </div>
            </div>
          </div>
        </form>
      </div>
      <div class="mt-8 bg-black rounded-2xl relative">
        <div class="absolute inset-0 flex justify-center items-center">
          <button @click="downloadPNG" v-show="downloadVisible" class="px-4 py-2 bg-pink-700 rounded-lg">Download</button>
        </div>
        <canvas ref="canvasRef" :width="userInputWidth" :height="userInputHeight" class="w-full rounded-2xl transition-all duration-150"></canvas>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, watch, onMounted } from 'vue';

  const userInputWidth = ref<number>(1920);
  const userInputHeight = ref<number>(1080);
  const selectedAspectRatio = ref('16:9');
  const canvasRef = ref<HTMLCanvasElement | null>(null);
  const downloadVisible = ref(false);
  
  let aspectRatio: number[] | null;
  
  let ctx: CanvasRenderingContext2D | null = null;

  let updatingWidth = false;
  let updatingHeight = false;

  watch(selectedAspectRatio, (newValue) => {
    if (newValue) {
      if (newValue !== 'custom') {
        const aspectRatio = newValue.split(':').map(Number);
        updatingHeight = true;
        userInputHeight.value = Math.round((userInputWidth.value / aspectRatio[0]) * aspectRatio[1]);
        updatingHeight = false;
      }
    }
    downloadVisible.value = false;
  });

  watch(userInputWidth, (newValue, oldValue) => {
    if (selectedAspectRatio.value && newValue !== oldValue && !updatingWidth) {
      if (selectedAspectRatio.value === 'custom') {
        downloadVisible.value = false;
        return;
      }
      const aspectRatio = selectedAspectRatio.value.split(':').map(Number);
      updatingHeight = true;
      userInputHeight.value = Math.round((newValue / aspectRatio[0]) * aspectRatio[1]);
      updatingHeight = false;
    }
    downloadVisible.value = false;
  }, { flush: 'sync' });

  watch(userInputHeight, (newValue, oldValue) => {
    if (selectedAspectRatio.value && newValue !== oldValue && !updatingHeight) {
      if (selectedAspectRatio.value === 'custom') {
        downloadVisible.value = false;
        return;
      }
      const aspectRatio = selectedAspectRatio.value.split(':').map(Number);
      updatingWidth = true;
      userInputWidth.value = Math.round((newValue / aspectRatio[1]) * aspectRatio[0]);
      updatingWidth = false;
    }
    downloadVisible.value = false;
  }, { flush: 'sync' });

  
  onMounted(() => {
    if (canvasRef.value) {
      ctx = canvasRef.value.getContext('2d');
    }
  });

  function updateAspectRatio() {
    const aspectRatioValue = selectedAspectRatio.value;
    if (aspectRatioValue === 'custom') {
      aspectRatio = null;
    } else {
      aspectRatio = aspectRatioValue.split(':').map(Number);
      userInputHeight.value = Math.round((userInputWidth.value / aspectRatio[0]) * aspectRatio[1]);
    }
  }

  function generateGuides() {
    if (ctx && canvasRef.value) {
      updateAspectRatio();
      const resolution = [userInputWidth.value, userInputHeight.value];
      ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
      drawGuides(resolution);
      downloadVisible.value = true;
    }
  }

  function drawGuides(resolution: number[]) {
    drawActionSafeGuide(resolution);
    drawTitleSafeGuide(resolution);
    drawCenterCutActionSafeGuide(resolution);
    drawCenterCutTitleSafeGuide(resolution);
  }

  function drawActionSafeGuide(resolution: number[]) {
    const width = resolution[0] * 0.93;
    const height = resolution[1] * 0.93;
    const offsetX = (resolution[0] - width) / 2;
    const offsetY = (resolution[1] - height) / 2;

    if (ctx) {
      ctx.strokeStyle = 'pink';
      ctx.lineWidth = 2;
      ctx.strokeRect(offsetX, offsetY, width, height);
    }
  }

  function drawTitleSafeGuide(resolution: number[]) {
    const width = resolution[0] * 0.9;
    const height = resolution[1] * 0.9;
    const offsetX = (resolution[0] - width) / 2;
    const offsetY = (resolution[1] - height) / 2;

    if (ctx) {
      ctx.strokeStyle = 'teal';
      ctx.lineWidth = 2;
      ctx.strokeRect(offsetX, offsetY, width, height);
    }
  }

  function drawCenterCutActionSafeGuide(resolution: number[]) {
    const aspect = [4, 3];
    const width = resolution[1] * (aspect[0] / aspect[1]) * 0.93;
    const height = resolution[1] * 0.93;
    const offsetX = (resolution[0] - width) / 2;
    const offsetY = (resolution[1] - height) / 2;

    if (ctx) {
      ctx.strokeStyle = 'pink';
      ctx.lineWidth = 2;
      ctx.setLineDash([5, 5]);
      ctx.beginPath();
      ctx.moveTo(offsetX, offsetY);
      ctx.lineTo(offsetX, offsetY + height);
      ctx.moveTo(offsetX + width, offsetY);
      ctx.lineTo(offsetX + width, offsetY + height);
      ctx.stroke();
      ctx.setLineDash([]);
    }
  }

  function drawCenterCutTitleSafeGuide(resolution: number[]) {
    const aspect = [4, 3];
    const width = resolution[1] * (aspect[0] / aspect[1]) * 0.9;
    const height = resolution[1] * 0.9;
    const offsetX = (resolution[0] - width) / 2;
    const offsetY = (resolution[1] - height) / 2;

    if (ctx) {
      ctx.strokeStyle = 'teal';
      ctx.lineWidth = 2;
      ctx.setLineDash([5, 5]);
      ctx.beginPath();
      ctx.moveTo(offsetX, offsetY);
      ctx.lineTo(offsetX, offsetY + height);
      ctx.moveTo(offsetX + width, offsetY);
      ctx.lineTo(offsetX + width, offsetY + height);
      ctx.stroke();
      ctx.setLineDash([]);
    }
  }

  function downloadPNG() {
    if (canvasRef.value) {
      const a = document.createElement('a');
      a.href = canvasRef.value.toDataURL('image/png');
      a.download = 'safe-area-guides.png';
      a.click();
    }
  }
  
</script>

<style scoped>
</style>
