<template>
  <div class="p-6 max-w-xl mx-auto bg-white border-1 border-gray-100 rounded-xl shadow-md space-y-6 relative">
    <h1 class="text-3xl font-bold text-center text-gray-800">🎯 Wheel Spinner</h1>

    <div class="flex gap-2">
      <input v-model="newItem" placeholder="Enter item"
        class="flex-1 px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400"
        @keyup.enter="addItem" />
      <button class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition" @click="addItem">
        Add
      </button>
    </div>

    <ul class="space-y-2">
      <li v-for="(item, index) in items" :key="index"
        class="flex justify-between items-center px-4 py-2 bg-gray-100 rounded-lg shadow-sm">
        <span>{{ item }}</span>
        <button class="text-red-500 hover:text-red-700" @click="removeItem(index)">
          &times;
        </button>
      </li>
    </ul>

    <div class="relative flex justify-center">
      <canvas ref="wheelCanvas" width="300" height="300" class="mb-4"></canvas>
      <div class="absolute top-[50%] left-[50%] -translate-x-1/2 -translate-y-[165px] rotate-[180deg]">
        <div
          class="w-0 h-0 border-l-[10px] border-l-transparent border-r-[10px] border-r-transparent border-b-[20px] border-b-red-600">
        </div>
      </div>
    </div>

    <div class="text-center">
      <button class="px-6 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600 transition" @click="spinWheel"
        :disabled="isSpinning">
        {{ isSpinning ? 'Spinning...' : 'Spin the Wheel' }}
      </button>
    </div>

    <div v-if="showResult" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg shadow-lg p-6 max-w-md text-center">
        <h2 class="text-2xl font-bold mb-4">🎉 Result</h2>
        <p class="text-lg">Selected: <strong>{{ selectedItem }}</strong></p>
        <div class="mt-4 space-x-2">
          <button class="px-4 py-2 bg-yellow-500 text-white rounded-lg hover:bg-yellow-600 transition"
            @click="keepItem">
            Keep
          </button>
          <button class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition"
            @click="removeSelectedItem">
            Remove
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';

const items = ref([]);
const newItem = ref('');
const selectedItem = ref(null);
const wheelCanvas = ref(null);
const angleOffset = ref(0);
const isSpinning = ref(false);
const showResult = ref(false);
let confettiInstance = null;

function addItem() {
  if (newItem.value.trim() !== '') {
    items.value.push(newItem.value.trim());
    newItem.value = '';
    drawWheel();
  }
}

function removeItem(index) {
  items.value.splice(index, 1);
  drawWheel();
}

function drawWheel() {
  const canvas = wheelCanvas.value;
  const ctx = canvas.getContext('2d');
  const total = items.value.length;
  const radius = canvas.width / 2;
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  if (total === 0) return;

  for (let i = 0; i < total; i++) {
    const startAngle = (2 * Math.PI / total) * i + angleOffset.value;
    const endAngle = startAngle + 2 * Math.PI / total;

    ctx.beginPath();
    ctx.moveTo(radius, radius);
    ctx.arc(radius, radius, radius, startAngle, endAngle);
    ctx.fillStyle = `hsl(${(i * 360 / total)}, 70%, 60%)`;
    ctx.fill();
    ctx.fillStyle = '#000';
    ctx.font = '14px sans-serif';
    ctx.textAlign = 'right';
    ctx.textBaseline = 'middle';
    ctx.save();
    ctx.translate(radius, radius);
    ctx.rotate(startAngle + Math.PI / total);
    ctx.fillText(items.value[i], radius - 10, 0);
    ctx.restore();
  }
}

function spinWheel() {
  if (items.value.length === 0 || isSpinning.value) return;

  isSpinning.value = true;
  selectedItem.value = null;
  showResult.value = false;

  const total = items.value.length;
  const duration = 3000;
  const fps = 60;
  const frames = (duration / 1000) * fps;
  let frame = 0;
  const startAngle = angleOffset.value;
  const endAngle = startAngle + (Math.PI * 6) + (2 * Math.PI * Math.random());

  function animate() {
    frame++;
    const progress = frame / frames;
    const easeOut = 1 - Math.pow(1 - progress, 3);
    angleOffset.value = startAngle + (endAngle - startAngle) * easeOut;
    drawWheel();

    if (frame < frames) {
      requestAnimationFrame(animate);
    } else {
      const finalAngle = angleOffset.value % (2 * Math.PI);
      const selectedIndex = Math.floor(total - (finalAngle / (2 * Math.PI)) * total) % total;
      selectedItem.value = items.value[selectedIndex];
      isSpinning.value = false;
      showResult.value = true;
      fireConfetti();
    }
  }

  animate();
}

function fireConfetti() {
  import('canvas-confetti').then((module) => {
    const confetti = module.default;
    confetti({
      particleCount: 150,
      spread: 70,
      origin: { y: 0.6 },
    });
  });
}

function keepItem() {
  showResult.value = false;
  selectedItem.value = null;
}

function removeSelectedItem() {
  const index = items.value.indexOf(selectedItem.value);
  if (index !== -1) {
    items.value.splice(index, 1);
  }
  showResult.value = false;
  selectedItem.value = null;
  drawWheel();
}

watch(items, drawWheel);
onMounted(drawWheel);
</script>

<style scoped>
canvas {
  border: 3px solid #ddd;
  border-radius: 50%;
  background: #f9f9f9;
  transition: transform 0.3s ease;
}
</style>
