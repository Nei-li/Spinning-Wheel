<template>
  <div class="app">
    <SpinningWheel
      :items="wheelItems"
      :duration="4000"
      title="🎡 Spinning Wheel - Wheel of Fortune"
      @onFinish="handleSpinFinish"
      ref="wheelRef"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import SpinningWheel from './components/SpinningWheel.vue'

const wheelRef = ref(null)

// Define wheel items with probability weights and prices
const wheelItems = ref([
  { label: 'Pencil ✏️', weight: 65, price: '$5' },
  { label: 'Diary 📓', weight: 8.75, price: '$15' },
  { label: 'Stickers 🎟️', weight: 8.75, price: '$8' },
  { label: 'Key Ring 🔑', weight: 8.75, price: '$12' },
  { label: 'Lanyard 🧵', weight: 8.75, price: '$10' }
])

// Handle spin completion
function handleSpinFinish(item) {
  console.log('Spin finished!', item)
}

// Optional: Programmatic control
function spinToSpecificItem(index) {
  if (wheelRef.value) {
    wheelRef.value.spinTo(index)
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.app {
width: 100%;
height: 100vh;

display: flex;
align-items: center;
justify-content: center;

background: linear-gradient(135deg, #667eea 0%), #764ba2 100%;
}
</style>
