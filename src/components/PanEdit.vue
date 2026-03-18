<!--
        PanEdit.vue
        -----------

        A custom specialized knob component for panning (-127 to 127).
-->
<script setup>
import { ref, computed, onUnmounted } from 'vue'

const props = defineProps({
  value: {
    type: Number,
    default: 0
  },
  opts: {
    type: Object,
    default: () => ({ min: -127, max: 127 })
  },
  lint: {
    type: Function,
    default: null
  },
  validate: {
    type: Function,
    default: null
  }
})

const emit = defineEmits(['change'])

const isDragging = ref(false)
const lastY = ref(0)
// Use a local ref for the "internal" numeric value during drag to avoid flickering
// and ensure we don't get stuck on sub-pixel movements due to Math.round
const internalValue = ref(props.value)

const rotation = computed(() => {
  const val = Number(props.value) || 0
  const min = props.opts.min ?? -127
  const max = props.opts.max ?? 127
  // Map min..max to -150..150 degrees
  const percent = (val - min) / (max - min)
  return (percent * 300) - 150
})

const onMouseDown = (e) => {
  isDragging.value = true
  lastY.value = e.clientY
  internalValue.value = Number(props.value) || 0
  
  window.addEventListener('mousemove', onMouseMove)
  window.addEventListener('mouseup', onMouseUp)
  document.body.style.cursor = 'ns-resize'
}

const onMouseMove = (e) => {
  if (!isDragging.value) return
  
  const deltaY = lastY.value - e.clientY
  const sensitivity = 0.5
  
  // Accumulate the change in our internal floating point value
  internalValue.value += deltaY * sensitivity
  
  // Constrain
  const min = props.opts.min ?? -127
  const max = props.opts.max ?? 127
  internalValue.value = Math.max(min, Math.min(max, internalValue.value))
  
  const roundedValue = Math.round(internalValue.value)
  
  if (roundedValue !== props.value) {
    emit('change', roundedValue)
  }
  
  lastY.value = e.clientY
}

const onMouseUp = () => {
  isDragging.value = false
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('mouseup', onMouseUp)
  document.body.style.cursor = ''
  
  if (props.validate) {
    props.validate(props.value)
  }
}

const onInputChange = (e) => {
  let val = parseInt(e.target.value)
  if (isNaN(val)) return // Don't emit if it's not a valid number yet (allow typing)
  
  const min = props.opts.min ?? -127
  const max = props.opts.max ?? 127
  val = Math.max(min, Math.min(max, val))
  
  emit('change', val)
}

onUnmounted(() => {
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('mouseup', onMouseUp)
})
</script>

<template>
  <div class="pan-edit">
    <div 
      class="knob-container" 
      @mousedown="onMouseDown"
      :title="`Value: ${value}`"
    >
      <div class="knob-outer">
        <div class="knob-inner" :style="{ transform: `rotate(${rotation}deg)` }">
          <div class="knob-indicator"></div>
        </div>
      </div>
    </div>
    <input 
      type="number" 
      :value="value" 
      @input="onInputChange"
      class="pan-input"
      :min="opts.min"
      :max="opts.max"
    />
  </div>
</template>

<style lang="scss" scoped>
.pan-edit {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 100%;

  .knob-container {
    cursor: ns-resize;
    user-select: none;
    flex-shrink: 0;
  }

  .knob-outer {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    background: #333;
    border: 2px solid #444;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: inset 0 2px 4px rgba(0,0,0,0.3);
  }

  .knob-inner {
    width: 100%;
    height: 100%;
    position: relative;
    transition: transform 0.05s linear;
  }

  .knob-indicator {
    position: absolute;
    top: 4px;
    left: 50%;
    width: 3px;
    height: 8px;
    background: #00abae;
    border-radius: 2px;
    transform: translateX(-50%);
  }

  .pan-input {
    width: 60px;
    padding: 6px 8px;
    border-radius: 4px;
    border: 1px solid #ccc;
    background-color: #fff;
    color: #333;
    font-size: 13px;
    outline: none;
    transition: border-color 0.2s;

    &:focus {
      border-color: #4caf50;
    }

    &::-webkit-inner-spin-button,
    &::-webkit-outer-spin-button {
      opacity: 1;
    }
  }
}
</style>
