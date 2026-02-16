<template>
  <div class="app">
    <UserForm 
      v-if="!showWheel"
      @formSubmitted="handleFormSubmit"
    />
    <SpinningWheel
      v-if="showWheel"
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
import UserForm from './components/UserForm.vue'

const wheelRef = ref(null)
const showWheel = ref(false) //no need to show wheel initially

//store form data
const userData = ref(null)

// Define wheel items with probability weights and prices
const wheelItems = ref([
  { label: 'Pencil ✏️', weight: 65, price: '$5' },
  { label: 'Diary 📓', weight: 8.75, price: '$15' },
  { label: 'Stickers 🎟️', weight: 8.75, price: '$8' },
  { label: 'Key Ring 🔑', weight: 8.75, price: '$12' },
  { label: 'Lanyard 🧵', weight: 8.75, price: '$10' }
])


//this is to handel form data
function handleFormSubmit(data) {
  userData.value = data
  showWheel.value = true
  console.log(showWheel.value)
}


// Handle spin completion
function handleSpinFinish(item) {
  console.log(userData.value) //this has all the data of user
  console.log(item) // This has the data of reward

  // add api call here
  //can make the function async and use await +api calll here

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
