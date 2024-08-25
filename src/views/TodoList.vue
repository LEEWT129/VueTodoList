<template>
  <div>
    <h1 class="title">TodoList</h1>
    <div class="todo-list">
      <div class="title-type">
        <button class="trash-btn" @click="openTrashModal">
          <div><i class="fa-regular fa-trash-can"></i> 資料回收桶</div>
        </button>
      </div>

      <div class="add-todo">
        <input v-model="newTodo" @keyup.enter="addTodo" placeholder="新增待辦事項" />
        <button class="icon-btn" @click="addTodo"><i class="fa-solid fa-plus"></i></button>
      </div>

      <ul>
        <li
          v-for="(todo, index) in todos"
          :key="index"
          draggable="true"
          @dragstart="dragStart(index)"
          @dragover.prevent
          @dragenter.prevent
          @drop="drop(index)"
        >
          <input type="checkbox" v-model="todo.completed" />
          <div>
            <span v-if="!todo.editing" :class="{ completed: todo.completed }">{{ todo.text }}</span>
            <input
              v-else
              v-model="todo.text"
              @keyup.enter="finishEdit(todo)"
              @blur="finishEdit(todo)"
            />
          </div>
          <div>
            <button class="icon-btn" v-if="!todo.editing" @click="startEdit(todo)">
              <i class="fa-regular fa-pen-to-square"></i>
            </button>
            <button class="icon-btn" v-else @click="finishEdit(todo)">
              <i class="fa-solid fa-check"></i>
            </button>
            <button class="icon-btn" @click="removeTodo(index)">
              <i class="fa-solid fa-x"></i>
            </button>
          </div>
        </li>
      </ul>
    </div>
    <TrashBin
      v-if="isTrashModalOpen"
      :trash="trash"
      @restore-item="restoreTodo"
      @close="closeTrashModal"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import TrashBin from '@/components/TrashBin.vue'

const newTodo = ref('')
const todos = ref([])
const draggedItemIndex = ref(null)
const trash = ref([])
const isTrashModalOpen = ref(false)

onMounted(() => {
  const savedTodos = localStorage.getItem('todos')
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos)
  }

  const savedTrash = localStorage.getItem('trash')
  if (savedTrash) {
    trash.value = JSON.parse(savedTrash)
  }
})

watch(
  todos,
  (newTodos) => {
    localStorage.setItem('todos', JSON.stringify(newTodos))
  },
  { deep: true }
)

watch(
  trash,
  (newTrash) => {
    localStorage.setItem('trash', JSON.stringify(newTrash))
  },
  { deep: true }
)

const openTrashModal = () => {
  isTrashModalOpen.value = true
}

const closeTrashModal = () => {
  isTrashModalOpen.value = false
}

const addToTrash = (item) => {
  trash.value.push(item)
}

const restoreTodo = (item) => {
  todos.value.unshift(item)
}

const removeTodo = (index) => {
  const removedItem = todos.value.splice(index, 1)[0]
  addToTrash(removedItem)
}

const addTodo = () => {
  if (newTodo.value.trim()) {
    todos.value.unshift({
      text: newTodo.value,
      completed: false,
      editing: false
    })
    newTodo.value = ''
  }
}

const dragStart = (index) => {
  draggedItemIndex.value = index
}

const drop = (index) => {
  const draggedItem = todos.value[draggedItemIndex.value]
  todos.value.splice(draggedItemIndex.value, 1)
  todos.value.splice(index, 0, draggedItem)
  draggedItemIndex.value = null
}

const startEdit = (todo) => {
  todo.editing = true
}

const finishEdit = (todo) => {
  todo.editing = false
  todo.completed = false
  todo.text = todo.text.trim()
  if (!todo.text) {
    removeTodo(todos.value.indexOf(todo))
  }
}
</script>

<style lang="scss" scoped>
.trash-btn {
  background-color: #fff;
  border: none;
}
ul {
  list-style-type: none;
  padding: 0;

  li {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
    cursor: move;

    input[type='checkbox'] {
      margin-right: 10px;
      align-self: center;
    }

    span,
    input[type='text'] {
      flex: 0 0 200px;
      width: 200px;
      min-height: 20px;
      overflow-wrap: break-word;
      word-wrap: break-word;
      word-break: break-all;
      white-space: normal;
      display: inline-block;
      align-items: center;
      vertical-align: middle;
    }

    .completed {
      text-decoration: line-through;
      color: #888;
    }
    .actions {
      display: flex;
      justify-content: center;
      margin-left: 10px;
      gap: 10px;
    }

    .icon-btn {
      align-items: center;
    }
  }
}

.add-todo {
  display: flex;
  margin-top: 10px;
  .icon-btn {
    margin-left: 10px;
  }
}
</style>
