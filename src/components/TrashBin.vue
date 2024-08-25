<template>
  <div class="modal-overlay" @click="closeModal">
    <div class="modal-content" @click.stop>
      <div><i class="fa-solid fa-trash-can"></i>垃圾桶</div>
      <ul v-if="trash.length > 0">
        <li v-for="(item, index) in trash" :key="index">
          <div class="item-container">
            <span>{{ item.text }}</span>
            <div class="buttons">
              <button class="icon-btn" @click="restoreItem(index)"><span>恢復項目</span></button>
              <button class="icon-btn" @click="permanentlyDelete(index)">
                <span>永久刪除</span>
              </button>
            </div>
          </div>
        </li>
      </ul>
      <div class="no-data-img">
        <img v-if="!trash.length" src="/nodata.png" alt="No Data" />
      </div>
      <button @click="closeModal" class="close-btn">關閉</button>
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  trash: {
    type: Array
  }
})

const emit = defineEmits(['restore-item', 'close'])

// 恢復項目
const restoreItem = (index) => {
  const restoredItem = props.trash.splice(index, 1)[0]
  emit('restore-item', restoredItem)
}

// 永久刪除項目
const permanentlyDelete = (index) => {
  props.trash.splice(index, 1)
}

const closeModal = () => {
  emit('close')
}
</script>

<style lang="scss" scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  .modal-content {
    background-color: #fff;
    padding: 20px;
    padding-bottom: 50px;
    border-radius: 8px;
    width: 300px;
    max-height: 80vh;
    position: relative;
    ul {
      max-height: 150px;
      overflow-y: auto;
    }
    .item-container {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    span {
      flex-grow: 1;
      text-align: left;
      width: 100px;
      white-space: normal;
      word-wrap: break-word;
    }
    .buttons {
      display: flex;
      gap: 10px;
    }
  }
}

.no-data-img {
  img {
    display: block;
    margin: 0 auto;
    width: 100px;
    height: auto;
  }
}

.close-btn {
  position: absolute;
  bottom: 10px;
  right: 10px;
  border: 1px solid rgb(66, 113, 165);
  border-radius: 5px;
  background-color: #fff;
  margin: 10px;
}

.close-btn:hover {
  background-color: rgb(66, 113, 165);
  color: #fff;
}
</style>
