<template>
  <div class="text-style-picker">
    <label>
      文字顏色:
      <input type="color" v-model="selectedColor" @input="updateStyle" />
    </label>
    <label class="font-size-label">
      文字大小:
      <div class="font-size-control">
        <input type="range" min="12" max="36" v-model="selectedSize" @input="updateStyle" />
        <span>{{ selectedSize }}px</span>
      </div>
    </label>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { defineEmits } from 'vue'

const emit = defineEmits(['updateStyle'])
const selectedColor = ref('#000000')
const selectedSize = ref(16)

const updateStyle = () => {
  const style = {
    color: selectedColor.value,
    fontSize: `${selectedSize.value}px`
  }
  emit('updateStyle', style)
}
</script>

<style scoped>
.text-style-picker {
  background-color: #ffffff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
  border: 1px solid #ddd;
}

.text-style-picker label {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
  font-size: 16px;
  color: #333;
}

.text-style-picker input[type='color'] {
  margin-top: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  width: 60px;
  height: 30px;
}

.font-size-label {
  display: flex;
  flex-direction: column;
}

.font-size-control {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin-top: 5px;
}

.text-style-picker input[type='range'] {
  cursor: pointer;
}

.text-style-picker span {
  font-weight: 700;
  color: #555;
  margin-top: 5px;
  font-size: 16px;
}
</style>
