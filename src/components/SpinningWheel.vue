<template>
  <div class="spinner-wheel-wrapper">
    <!-- Title -->
    <h1 class="wheel-title">{{ title }}</h1>

    <!-- Main Wheel Container -->
    <div class="wheel-container">
      <!-- SVG Wheel -->
      <svg
        class="wheel-svg"
        :style="{ transform: `rotate(${rotation}deg)` }"
        viewBox="0 0 400 400"
        width="200"
        height="200"
      >
        <!-- Slices -->
        <g v-for="(slice, index) in slices" :key="`slice-${index}`">
          <!-- Slice Path -->
          <path
            :d="slice.path"
            :fill="slice.item === lastResult ? '#9B59B6' : slice.color"
            stroke="white"
            :stroke-width="slice.item === lastResult ? 6 : 2"
          />
          <!-- Slice Label -->
          <text
            :x="slice.labelX"
            :y="slice.labelY"
            :transform="`rotate(${slice.label}deg ${200} ${200})`"
            class="slice-label"
            text-anchor="middle"
            dominant-baseline="middle"
          >
            {{ slice.item.label || slice.item }}
          </text>
        </g>

        <!-- Center Circle with Spin Button -->
        <g>
          <circle cx="200" cy="200" r="45" fill="#FFD700" stroke="white" stroke-width="3" />
          <circle cx="200" cy="200" r="35" fill="#FF6B9D" />
          <text
            x="200"
            y="200"
            class="spin-text"
            text-anchor="middle"
            dominant-baseline="middle"
            @click="spin"
            :style="{ cursor: isSpinning ? 'not-allowed' : 'pointer', opacity: isSpinning ? 0.6 : 1 }"
          >
            🎡
          </text>
        </g>
      </svg>

      <!-- Confetti -->
      <canvas
        v-if="showConfetti"
        ref="confettiCanvas"
        class="confetti-canvas"
      ></canvas>
    </div>

    <!-- Spin Button -->
    <button
      class="spin-button"
      @click="spin"
      :disabled="isSpinning"
    >
      <span v-if="!isSpinning">🎡 SPIN</span>
      <span v-else>⏳ SPINNING...</span>
    </button>

    <!-- Result Display -->
    <transition name="result-fade">
      <div v-if="lastResult" class="result-display">
        <h2 class="result-text">🎉 You Won!</h2>
      </div>
    </transition>


  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// Props
const props = defineProps({
  items: {
    type: Array,
    required: true,
    validator: (arr) => arr.length > 0
  },
  duration: {
    type: Number,
    default: 2000
  },
  title: {
    type: String,
    default: 'Spinning Wheel - Wheel of Fortune'
  },
  autoSpin: {
    type: Boolean,
    default: false
  }
})

// Emits
const emit = defineEmits(['onFinish'])

// State
const rotation = ref(0)
const isSpinning = ref(false)
const lastResult = ref(null)
const showConfetti = ref(false)
const confettiCanvas = ref(null)
const animationFrameId = ref(null)
const spinCount = ref(0)

// Constants
const SLICE_COUNT = props.items.length
const FULL_ROTATION = 360
const MIN_ROTATIONS = 5

// Color palette - vibrant and distinct
const COLORS = [
  '#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8',
  '#F7DC6F', '#BB8FCE', '#85C1E2', '#F8B88B', '#A9DFBF',
  '#F48FB1', '#000000', '#F7DC6F', '#BB8FCE', '#000000'
]

// Get color for slice
function getSliceColor(index) {
  return COLORS[index % COLORS.length]
}

// Calculate SVG paths and positions for slices
const slices = computed(() => {
  // Calculate total weight for percentage calculation
  const totalWeight = props.items.reduce((sum, item) => {
    const weight = typeof item === 'object' ? (item.weight || 1) : 1
    return sum + weight
  }, 0)

  return props.items.map((item, index) => {
    const sliceAngle = FULL_ROTATION / SLICE_COUNT
    const startAngle = (index * sliceAngle - 90) * (Math.PI / 180)
    const endAngle = ((index + 1) * sliceAngle - 90) * (Math.PI / 180)
    const radius = 180

    // Calculate path corners
    const x1 = 200 + radius * Math.cos(startAngle)
    const y1 = 200 + radius * Math.sin(startAngle)
    const x2 = 200 + radius * Math.cos(endAngle)
    const y2 = 200 + radius * Math.sin(endAngle)

    // SVG arc path
    const largeArc = sliceAngle > 180 ? 1 : 0
    const path = `M 200 200 L ${x1} ${y1} A ${radius} ${radius} 0 ${largeArc} 1 ${x2} ${y2} Z`

    // Label position (middle of slice, at 2/3 distance from center)
    const labelAngle = startAngle + (endAngle - startAngle) / 2
    const labelRadius = radius * 0.65
    const labelX = 200 + labelRadius * Math.cos(labelAngle)
    const labelY = 200 + labelRadius * Math.sin(labelAngle)
    const labelRotation = (index * sliceAngle) % 360

    // Calculate probability percentage
    const weight = typeof item === 'object' ? (item.weight || 1) : 1
    const probability = ((weight / totalWeight) * 100).toFixed(0)

    return {
      item,
      path,
      color: getSliceColor(index),
      labelX,
      labelY,
      label: labelRotation,
      probability
    }
  })
})

// Fair weighted random selection based on probability
function getRandomIndex() {
  // Check if items have weight property (for weighted probability)
  const hasWeights = props.items.some(item => typeof item === 'object' && item.weight !== undefined)
  
  if (hasWeights) {
    // Weighted selection
    const totalWeight = props.items.reduce((sum, item) => {
      const weight = typeof item === 'object' ? (item.weight || 1) : 1
      return sum + weight
    }, 0)
    
    let random = Math.random() * totalWeight
    for (let i = 0; i < props.items.length; i++) {
      const weight = typeof props.items[i] === 'object' ? (props.items[i].weight || 1) : 1
      if (random < weight) {
        return i
      }
      random -= weight
    }
    return props.items.length - 1
  } else {
    // Equal probability for all items
    return Math.floor(Math.random() * SLICE_COUNT)
  }
}

// Spin to specific index
function spinTo(targetIndex) {
  if (isSpinning.value) return

  isSpinning.value = true
  lastResult.value = null
  showConfetti.value = false

  // Calculate duration based on spin count
  // First spin: 4000ms (slow), then gets faster
  let duration = props.duration
  if (spinCount.value > 0) {
    // Decrease duration by 200ms for each spin, minimum 1500ms
    duration = Math.max(props.duration - (spinCount.value * 200), 1500)
  }

  const sliceAngle = FULL_ROTATION / SLICE_COUNT
  const randomRotations = MIN_ROTATIONS + Math.random() * 2

  // Calculate the exact rotation needed
  // Pointer is at top (0°), so we need the target slice to be at 0°
  // Slice index 0 starts at -90°, so adjust accordingly
  const targetAngle = targetIndex * sliceAngle
  const finalRotation = randomRotations * FULL_ROTATION + (FULL_ROTATION - targetAngle)

  animateWheel(finalRotation, duration)

  setTimeout(() => {
    isSpinning.value = false
    lastResult.value = props.items[targetIndex]
    emit('onFinish', props.items[targetIndex])
    showConfetti.value = true
    triggerConfetti()
    spinCount.value++ // Increment spin counter
  }, duration)
}

// Main spin function - random selection
function spin() {
  const targetIndex = getRandomIndex()
  spinTo(targetIndex)
}

// Animate wheel rotation
function animateWheel(targetRotation, duration) {
  const startRotation = rotation.value
  const startTime = Date.now()

  const animate = () => {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)

    // Cubic bezier easing for smooth deceleration
    const easeProgress = 1 - Math.pow(1 - progress, 3)

    rotation.value = startRotation + (targetRotation - startRotation) * easeProgress

    if (progress < 1) {
      animationFrameId.value = requestAnimationFrame(animate)
    }
  }

  animate()
}

// Confetti animation with balloons, coins, and gems bursting out
function triggerConfetti() {
  if (!confettiCanvas.value) return

  const canvas = confettiCanvas.value
  const ctx = canvas.getContext('2d')
  const { width, height } = canvas.getBoundingClientRect()

  canvas.width = width
  canvas.height = height

  // Center of the wheel (approximately where it is positioned)
  const centerX = width / 2
  const centerY = height / 3

  // Create particles - mix of regular confetti and coins/gems bursting from center
  const particles = Array.from({ length: 100 }, (_, i) => {
    let particle
    
    if (i < 30) {
      // Coins and gems bursting from wheel center
      const angle = (i / 30) * Math.PI * 2 + (Math.random() - 0.5) * 0.5
      const speed = Math.random() * 8 + 10
      const distance = Math.random() * 50 + 30

      particle = {
        x: centerX + Math.cos(angle) * distance,
        y: centerY + Math.sin(angle) * distance,
        vx: Math.cos(angle) * speed,
        vy: Math.sin(angle) * speed,
        size: Math.random() * 8 + 10,
        color: Math.random() > 0.5 ? '#FFD700' : '#FF69B4', // Gold or Pink
        rotation: Math.random() * Math.PI * 2,
        rotationSpeed: (Math.random() - 0.5) * 0.25,
        type: Math.random() > 0.5 ? 'coin' : 'gem'
      }
    } else {
      // Regular falling confetti
      const type = Math.random()
      particle = {
        x: Math.random() * width,
        y: -20,
        vx: (Math.random() - 0.5) * 12,
        vy: Math.random() * 6 + 3,
        size: Math.random() * 10 + 6,
        color: COLORS[Math.floor(Math.random() * COLORS.length)],
        rotation: Math.random() * Math.PI * 2,
        rotationSpeed: (Math.random() - 0.5) * 0.15,
        type: type < 0.4 ? 'balloon' : type < 0.7 ? 'star' : 'square'
      }
    }

    return particle
  })

  let frameCount = 0
  const maxFrames = 180

  const drawBalloon = (x, y, size, color) => {
    ctx.fillStyle = color
    ctx.beginPath()
    ctx.arc(x, y, size / 2, 0, Math.PI * 2)
    ctx.fill()
    // Balloon string
    ctx.strokeStyle = color
    ctx.lineWidth = 1
    ctx.beginPath()
    ctx.moveTo(x, y + size / 2)
    ctx.lineTo(x, y + size / 2 + 15)
    ctx.stroke()
  }

  const drawStar = (x, y, size, color) => {
    ctx.save()
    ctx.translate(x, y)
    ctx.rotate(Math.random() * Math.PI)
    ctx.fillStyle = color
    ctx.beginPath()
    for (let i = 0; i < 5; i++) {
      const angle = (i * 4 * Math.PI) / 5 - Math.PI / 2
      const x1 = Math.cos(angle) * size
      const y1 = Math.sin(angle) * size
      if (i === 0) ctx.moveTo(x1, y1)
      else ctx.lineTo(x1, y1)
    }
    ctx.closePath()
    ctx.fill()
    ctx.restore()
  }

  const drawCoin = (x, y, size, color) => {
    ctx.save()
    ctx.translate(x, y)
    ctx.fillStyle = color
    ctx.beginPath()
    ctx.arc(0, 0, size / 2, 0, Math.PI * 2)
    ctx.fill()
    // Coin outline and shine
    ctx.strokeStyle = '#FFA500'
    ctx.lineWidth = 2
    ctx.stroke()
    // Inner detail
    ctx.fillStyle = 'rgba(255,255,255,0.3)'
    ctx.beginPath()
    ctx.arc(-size / 6, -size / 6, size / 8, 0, Math.PI * 2)
    ctx.fill()
    ctx.restore()
  }

  const drawGem = (x, y, size, color) => {
    ctx.save()
    ctx.translate(x, y)
    ctx.rotate(frameCount * 0.05)
    ctx.fillStyle = color
    ctx.beginPath()
    ctx.moveTo(0, -size / 2)
    ctx.lineTo(size / 2, 0)
    ctx.lineTo(size / 3, size / 2)
    ctx.lineTo(-size / 3, size / 2)
    ctx.lineTo(-size / 2, 0)
    ctx.closePath()
    ctx.fill()
    // Gem shine
    ctx.strokeStyle = 'rgba(255,255,255,0.6)'
    ctx.lineWidth = 1
    ctx.stroke()
    ctx.restore()
  }

  const drawConfetti = () => {
    ctx.clearRect(0, 0, width, height)

    particles.forEach((particle) => {
      if (particle.type === 'coin' || particle.type === 'gem') {
        // Coins and gems expire faster and have gravity
        particle.vy += 0.3 // stronger gravity for coins/gems
      } else {
        particle.vy += 0.15 // normal gravity for confetti
      }
      particle.y += particle.vy
      particle.x += particle.vx
      particle.rotation += particle.rotationSpeed

      // Flashing effect
      const flash = Math.sin(frameCount * 0.1) * 0.5 + 0.5
      ctx.globalAlpha = (1 - frameCount / maxFrames) * (0.5 + flash * 0.5)

      if (particle.type === 'balloon') {
        drawBalloon(particle.x, particle.y, particle.size, particle.color)
      } else if (particle.type === 'star') {
        drawStar(particle.x, particle.y, particle.size / 2, particle.color)
      } else if (particle.type === 'coin') {
        drawCoin(particle.x, particle.y, particle.size, particle.color)
      } else if (particle.type === 'gem') {
        drawGem(particle.x, particle.y, particle.size, particle.color)
      } else {
        ctx.save()
        ctx.translate(particle.x, particle.y)
        ctx.rotate(particle.rotation)
        ctx.fillStyle = particle.color
        ctx.fillRect(-particle.size / 2, -particle.size / 2, particle.size, particle.size)
        ctx.restore()
      }
    })

    frameCount++
    if (frameCount < maxFrames) {
      requestAnimationFrame(drawConfetti)
    } else {
      showConfetti.value = false
    }
  }

  drawConfetti()
}

// Reset state
function reset() {
  lastResult.value = null
  showConfetti.value = false
  spinCount.value = 0
}

// Auto spin on mount if enabled
onMounted(() => {
  if (props.autoSpin) {
    setTimeout(() => spin(), 500)
  }
})

// Cleanup
onUnmounted(() => {
  if (animationFrameId.value) {
    cancelAnimationFrame(animationFrameId.value)
  }
})

// Expose method for programmatic control
defineExpose({
  spin,
  spinTo,
  reset
})
</script>

<style scoped>
/* Container */
.spinner-wheel-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 40px 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: white;
}

.wheel-title {
  font-size: 1.8em;
  margin-bottom: 40px;
  text-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  letter-spacing: 2px;
  text-align: center;
}

/* Wheel Container */
.wheel-container {
  position: relative;
  width: 500px;
  height: 500px;
  margin-bottom: 50px;
}

.pointer {
  position: absolute;
  top: -12px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 23px solid transparent;
  border-right: 23px solid transparent;
  border-top: 45px solid #FF6B9D;
  z-index: 20;
  filter: drop-shadow(0 4px 10px rgba(0, 0, 0, 0.4));
}

.pointer::after {
  content: '';
  position: absolute;
  top: -45px;
  left: -23px;
  width: 45px;
  height: 45px;
  background: radial-gradient(circle, rgba(255, 107, 157, 0.3), transparent);
  border-radius: 50%;
  filter: blur(10px);
}

/* SVG Wheel */
.wheel-svg {
  width: 100%;
  height: 100%;
  filter: drop-shadow(0 0 50px rgba(255, 215, 0, 0.6)) drop-shadow(0 0 100px rgba(102, 126, 234, 0.3));
  transition: transform 0.05s linear;
}

.slice-label {
  font-size: 16px;
  font-weight: 900;
  fill: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  font-family: 'Segoe UI', sans-serif;
  pointer-events: none;
}

.slice-probability {
  font-size: 12px;
  font-weight: 700;
  fill: white;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);
  font-family: 'Segoe UI', sans-serif;
  pointer-events: none;
  opacity: 0.9;
}

/* Confetti Canvas */
.confetti-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

/* Spin Text in Center */
.spin-text {
  font-size: 48px;
  cursor: pointer;
  transition: all 0.3s ease;
  filter: drop-shadow(0 2px 5px rgba(0, 0, 0, 0.3));
}

.spin-text:hover {
  font-size: 55px;
  filter: drop-shadow(0 4px 10px rgba(0, 0, 0, 0.5));
}

.spin-text:active {
  font-size: 45px;
}

/* Buttons */
.spin-button {
  padding: 12px 30px;
  font-size: 16px;
  font-weight: bold;
  border: none;
  border-radius: 30px;
  background: linear-gradient(135deg, #FF6B6B 0%, #FF9F43 100%);
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.3);
  text-transform: uppercase;
  letter-spacing: 2px;
  margin-bottom: 30px;
}

.spin-button:hover:not(:disabled) {
  transform: scale(1.1);
  box-shadow: 0 12px 40px rgba(255, 107, 107, 0.5);
}

.spin-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.reset-button {
  padding: 12px 40px;
  font-size: 16px;
  font-weight: bold;
  border: 2px solid white;
  border-radius: 30px;
  background: transparent;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.reset-button:hover {
  background: white;
  color: #667eea;
  transform: scale(1.05);
}

/* Result Display */
.result-display {
  text-align: center;
  margin-top: 20px;
  animation: win-animation 0.8s cubic-bezier(0.34, 1.56, 0.64, 1), shake-screen 0.5s ease-in-out;
}

.result-text {
  font-size: 2.5em;
  margin: 0 0 15px 0;
  text-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.result-item {
  font-size: 2em;
  font-weight: bold;
  color: #FFD700;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  margin: 0;
}

.result-price {
  font-size: 1.5em;
  font-weight: bold;
  color: #4ECDC4;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  margin: 10px 0 0 0;
}

/* Bounce Animation for Winner */
.bounce-animation {
  animation: bounce-winner 0.6s ease-in-out;
}

@keyframes bounce-winner {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

/* Transitions */
.winner-appear-enter-active,
.winner-appear-leave-active {
  transition: all 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.winner-appear-enter-from,
.winner-appear-leave-to {
  opacity: 0;
  transform: scale(0.3) rotateZ(-30deg);
}

.winner-appear-enter-to {
  animation: pop-in 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes pop-in {
  0% {
    transform: scale(0.3) rotateZ(-30deg);
    opacity: 0;
  }
  50% {
    transform: scale(1.15) rotateZ(5deg);
  }
  100% {
    transform: scale(1) rotateZ(0deg);
    opacity: 1;
  }
}

@keyframes win-animation {
  0% {
    transform: scale(0) rotateX(0deg);
    opacity: 0;
  }
  50% {
    transform: scale(1.1) rotateX(10deg);
  }
  100% {
    transform: scale(1) rotateX(0deg);
    opacity: 1;
  }
}

@keyframes pulse-win {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.15);
  }
}

@keyframes shake-screen {
  0%, 100% {
    transform: translateX(0);
  }
  10% {
    transform: translateX(-8px);
  }
  20% {
    transform: translateX(8px);
  }
  30% {
    transform: translateX(-8px);
  }
  40% {
    transform: translateX(8px);
  }
  50% {
    transform: translateX(-5px);
  }
  60% {
    transform: translateX(5px);
  }
  70% {
    transform: translateX(-3px);
  }
  80% {
    transform: translateX(3px);
  }
  90% {
    transform: translateX(-1px);
  }
}

/* Responsive */
@media (max-width: 1024px) {
  .wheel-container {
    width: 400px;
    height: 400px;
  }

  .wheel-svg {
    width: 100%;
    height: 100%;
  }

  .spin-button {
    padding: 12px 35px;
    font-size: 16px;
  }
}

@media (max-width: 768px) {
  .spinner-wheel-wrapper {
    padding: 20px;
    min-height: auto;
  }

  .wheel-title {
    font-size: 1.5em;
    margin-bottom: 20px;
  }

  .wheel-container {
    width: 280px;
    height: 280px;
    margin-bottom: 20px;
  }

  .spin-button {
    padding: 12px 40px;
    font-size: 16px;
    margin-bottom: 20px;
  }

  .result-text {
    font-size: 1.8em;
  }
}

@media (max-width: 600px) {
  .spinner-wheel-wrapper {
    padding: 15px;
    min-height: 100vh;
  }

  .wheel-title {
    font-size: 1.3em;
    margin-bottom: 15px;
  }

  .wheel-container {
    width: 250px;
    height: 250px;
    margin-bottom: 20px;
  }

  .pointer {
    top: -12px;
    border-left: 23px solid transparent;
    border-right: 23px solid transparent;
    border-top: 45px solid #FF6B9D;
  }

  .pointer::after {
    width: 45px;
    height: 45px;
    top: -45px;
    left: -23px;
  }

  .spin-button {
    padding: 10px 30px;
    font-size: 14px;
    margin-bottom: 15px;
  }

  .result-text {
    font-size: 1.5em;
  }

  .result-item {
    font-size: 1.2em;
  }

  .reset-button {
    padding: 8px 25px;
    font-size: 13px;
  }
}

@media (max-width: 400px) {
  .wheel-title {
    font-size: 1.1em;
    margin-bottom: 10px;
  }

  .wheel-container {
    width: 220px;
    height: 220px;
    margin-bottom: 15px;
  }

  .spin-button {
    padding: 8px 20px;
    font-size: 12px;
  }

  .result-text {
    font-size: 1.3em;
  }
}
</style>
