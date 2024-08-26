<template>
  <div class="saved-list">
    <ul>
      <li v-for="(state, index) in savedStates" :key="index" @click="$emit('select-state', index)">
        <span v-if="state">
          {{ state.title }}
        </span>
        <button class="icon-btn" @click.stop="deleteState(index)">
          <i class="fa-solid fa-x"></i>
        </button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  savedStates: Array,
  default: () => []
})

const emit = defineEmits(['select-state', 'delete-state'])

const deleteState = (index) => {
  emit('delete-state', index)
}
</script>

<style lang="scss" scoped>
.saved-list {
  width: 150px;

  ul {
    list-style-type: none;
    padding: 0;
    margin: 0;

    li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      cursor: pointer;
      padding: 5px;
      margin-bottom: 5px;
      border: 1px solid #ccc;
      width: 100%;

      span {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        flex: 1;
      }

      .icon-btn {
        background: none;
        border: none;
        cursor: pointer;
        padding: 0;
        margin-left: 10px;
      }
    }

    li:hover {
      background-color: #f0f0f0;
    }
  }
}
</style>
