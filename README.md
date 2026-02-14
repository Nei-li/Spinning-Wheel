# 🎡 Spinning Wheel - Vue 3 Component

A fully functional, production-ready Spinner Wheel (Wheel of Fortune) component built with Vue 3 Composition API. Features SVG-based rendering, smooth animations, confetti effects, and comprehensive customization options.

## ✨ Features

✅ **SVG-Based Rendering** - Smooth, scalable graphics (not canvas)
✅ **Composition API** - Built with `<script setup>` syntax
✅ **Dynamic Items** - Accept any number of items
✅ **Smooth Animations** - Cubic-bezier easing with configurable duration
✅ **Fair Selection** - Random selection logic
✅ **Confetti Effect** - Built-in confetti animation on win
✅ **Programmatic Control** - `spinTo(index)` method for specific outcomes
✅ **Event Emission** - `onFinish` event with selected item
✅ **Responsive Design** - Works on mobile and desktop
✅ **No External Dependencies** - Except Vue 3
✅ **Click Prevention** - Prevents multiple spins simultaneously
✅ **Pointer Indicator** - Fixed top pointer to show selection
✅ **Auto Spin** - Optional auto-spin on mount
✅ **Reset Capability** - Reset wheel state

## 📦 Installation

```bash
npm install
npm run dev
```

## 🚀 Basic Usage

### Simple String Items

```vue
<template>
  <SpinningWheel
    :items="['Prize 1', 'Prize 2', 'Prize 3']"
    @onFinish="handleFinish"
  />
</template>

<script setup>
import SpinningWheel from './components/SpinningWheel.vue'

function handleFinish(item) {
  console.log('Won:', item)
}
</script>
```

### Object Items with Labels

```vue
<template>
  <SpinningWheel
    :items="prizes"
    :duration="5000"
    title="Win Amazing Prizes!"
    @onFinish="handleFinish"
    ref="wheel"
  />
</template>

<script setup>
import SpinningWheel from './components/SpinningWheel.vue'
import { ref } from 'vue'

const wheel = ref(null)

const prizes = ref([
  { label: '$50' },
  { label: '$100' },
  { label: '$250' },
  { label: '$500' },
  { label: '$1000' },
  { label: 'Free Spin' },
  { label: 'Jackpot!' }
])

function handleFinish(item) {
  console.log('You won:', item.label)
}
</script>
```

## 📋 Props

| Prop | Type | Default | Required | Description |
|------|------|---------|----------|-------------|
| `items` | Array | - | ✅ | Array of strings or objects with `label` property |
| `duration` | Number | 4000 | ❌ | Spin animation duration in milliseconds |
| `title` | String | '🎡 Spinning Wheel' | ❌ | Wheel title display |
| `autoSpin` | Boolean | false | ❌ | Auto-spin wheel on component mount |

## 🎯 Events

### onFinish
Emitted when the spin animation completes.

```vue
<SpinningWheel
  :items="items"
  @onFinish="handleSpinComplete"
/>
```

```javascript
function handleSpinComplete(item) {
  // item is the selected item from the items array
  console.log('Selected:', item)
}
```

## 🎮 Methods

### spin()
Trigger a random spin.

```vue
<template>
  <button @click="wheel.spin()">Spin</button>
  <SpinningWheel ref="wheel" :items="items" />
</template>

<script setup>
const wheel = ref(null)
</script>
```

### spinTo(index)
Spin to a specific item by index (0-based).

```javascript
// Spin to always land on index 0
wheel.value.spinTo(0)

// Programmatically select a winner
const winnerIndex = Math.floor(Math.random() * items.value.length)
wheel.value.spinTo(winnerIndex)
```

### reset()
Reset wheel state and clear results.

```javascript
wheel.value.reset()
```

## 🛠️ Advanced Usage

### Programmatic Control Example

```vue
<template>
  <div>
    <SpinningWheel
      :items="prizes"
      @onFinish="onSpinComplete"
      ref="wheelComponent"
    />
    
    <button @click="spinRandomly">Random Spin</button>
    <button @click="spinToJackpot">Win Jackpot!</button>
    <button @click="resetWheel">Reset</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import SpinningWheel from './components/SpinningWheel.vue'

const wheelComponent = ref(null)

const prizes = ref([
  { label: '$10' },
  { label: '$25' },
  { label: '$50' },
  { label: '$100' },
  { label: '$500' },
  { label: 'Free Spin' },
  { label: 'JACKPOT!' }
])

function spinRandomly() {
  wheelComponent.value.spin()
}

function spinToJackpot() {
  // JACKPOT is at index 6
  wheelComponent.value.spinTo(6)
}

function resetWheel() {
  wheelComponent.value.reset()
}

function onSpinComplete(item) {
  setTimeout(() => {
    alert(`🎉 Congratulations! You won: ${item.label}`)
  }, 500)
}
</script>
```

### Auto Spin on Mount

```vue
<SpinningWheel
  :items="prizes"
  autoSpin
  @onFinish="handleResult"
/>
```

### Custom Duration

```vue
<SpinningWheel
  :items="items"
  :duration="6000"
  title="Quick Spin!"
/>
```

## 🎨 Customization

### Colors
The component uses a built-in color palette. To customize, edit the `COLORS` array in `SpinningWheel.vue`:

```javascript
const COLORS = [
  '#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8',
  '#F7DC6F', '#BB8FCE', '#85C1E2', '#F8B88B', '#A9DFBF',
  // ... add more colors as needed
]
```

### Styling
The component uses scoped styles. Override default styles:

```vue
<style scoped>
:deep(.wheel-title) {
  font-size: 4em;
  color: gold;
}
</style>
```

## 📐 Component Structure

```
SpinningWheel.vue
├── template
│   ├── wheel-container (main wrapper)
│   ├── pointer (fixed indicator)
│   ├── SVG wheel
│   │   ├── slices (segments with labels)
│   │   └── center circle
│   ├── spin button
│   ├── result display
│   ├── reset button
│   └── confetti canvas
└── script setup
    ├── props (items, duration, title, autoSpin)
    ├── emits (onFinish)
    ├── state (rotation, isSpinning, etc.)
    ├── computed (slices)
    ├── methods (spin, spinTo, reset, etc.)
    └── lifecycle (onMounted, onUnmounted)
```

## 🎬 Animation Details

- **Minimum Rotations**: 5 full rotations before stopping
- **Easing**: Cubic-bezier (smooth deceleration)
- **Duration**: Configurable (default 4000ms)
- **Confetti**: 50 particles, 120 frame animation

## 📱 Responsive Behavior

The component automatically adjusts for smaller screens:
- Wheel size: 450px → 300px
- Button size: Scales proportionally
- Text size: Responsive scaling
- Touch-friendly on mobile

## ⚡ Performance

- Smooth 60 FPS animation using `requestAnimationFrame`
- SVG rendering for scalability
- Canvas confetti with proper cleanup
- No layout thrashing

## 🔒 Browser Support

- Chrome/Edge 88+
- Firefox 87+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Mobile)

## 📝 Example Files

### App.vue
Shows complete example with 7 prize items ($5-$100).

### SpinningWheel.vue
Complete, self-contained component with all features.

## 🎓 Learning Resources

The component demonstrates:
- Vue 3 Composition API with `<script setup>`
- SVG path generation
- Canvas animations (confetti)
- CSS animations and transitions
- Responsive design patterns
- Event emission and props
- Method exposure with `defineExpose`

## 🐛 Troubleshooting

**Wheel not spinning?**
- Check if component is disabled
- Verify items array is not empty

**Items not showing?**
- Ensure items have `label` property or are strings
- Check browser console for errors

**Confetti not showing?**
- Canvas reference must be valid
- Check browser permissions

## 📄 License

MIT License - Feel free to use in commercial projects.

## 🤝 Contributing

Suggestions and improvements welcome!

---

Built with ❤️ using Vue 3 Composition API
