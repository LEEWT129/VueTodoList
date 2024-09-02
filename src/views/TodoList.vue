<template>
  <div class="container">
    <div class="main-content">
      <div class="style-picker-container">
        <TextStylePicker @update-style="updateTodoStyle" />
      </div>
      <div class="todo-section">
        <input v-model="title" @blur="saveTitle" class="title-input" />
        <div class="todo-list">
          <div class="title-type">
            <button class="trash-btn" @click="openTrashModal">
              <div><i class="fa-regular fa-trash-can"></i> 資料回收桶</div>
            </button>
            <button class="save-btn" @click="saveCurrentState">
              <i class="fa-solid fa-save"></i> 保存
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
              :style="todo.style"
              draggable="true"
              @dragstart="dragStart(index)"
              @dragover.prevent
              @dragenter.prevent
              @drop="drop(index)"
            >
              <input type="checkbox" v-model="todo.completed" />
              <div>
                <span v-if="!todo.editing" :class="{ completed: todo.completed }">
                  {{ todo.text }}
                </span>
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
      </div>

      <div class="saved-list-section">
        <SavedList
          :savedStates="savedStates"
          @select-state="loadState"
          @delete-state="removeSavedState"
        />
      </div>
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
import SavedList from '@/components/SavedList.vue'
import TextStylePicker from '@/components/TextStylePicker.vue'

const title = ref('TodoList')
const newTodo = ref('')
const todos = ref([])
const savedStates = ref([])
const draggedItemIndex = ref(null)
const trash = ref([])
const isTrashModalOpen = ref(false)
const currentSavedStateIndex = ref(null)

onMounted(() => {
  const savedTitle = localStorage.getItem('title')
  if (savedTitle) {
    title.value = savedTitle
  }

  const savedTodos = localStorage.getItem('todos')
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos)
  }

  const savedTrash = localStorage.getItem('trash')
  if (savedTrash) {
    trash.value = JSON.parse(savedTrash)
  }

  const savedStatesFromStorage = localStorage.getItem('savedStates')
  if (savedStatesFromStorage) {
    savedStates.value = JSON.parse(savedStatesFromStorage)
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

watch(
  savedStates,
  (newSavedStates) => {
    localStorage.setItem('savedStates', JSON.stringify(newSavedStates))
  },
  { deep: true }
)

const saveTitle = () => {
  localStorage.setItem('title', title.value)
}

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

const saveCurrentState = () => {
  const state = {
    title: title.value,
    todos: JSON.parse(JSON.stringify(todos.value))
  }

  // 查找是否有相同標題
  const existingIndex = savedStates.value.findIndex((s) => s.title === title.value)

  if (existingIndex !== -1) {
    savedStates.value[existingIndex] = state
  } else {
    savedStates.value.push(state)
  }

  todos.value = []
  title.value = 'TodoList'
  newTodo.value = ''
  currentSavedStateIndex.value = null
}

const removeSavedState = (index) => {
  savedStates.value.splice(index, 1)
  if (currentSavedStateIndex.value === index) {
    currentSavedStateIndex.value = null
  }
}

const loadState = (index) => {
  const state = savedStates.value[index]
  title.value = state.title
  todos.value = JSON.parse(JSON.stringify(state.todos))
  currentSavedStateIndex.value = index
}

const updateTodoStyle = (style) => {
  todos.value.forEach((todo) => {
    if (!todo.style) {
      todo.style = {}
    }
    Object.assign(todo.style, style)
  })
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  justify-content: center;
  padding: 20px;
}

.main-content {
  display: flex;
  align-items: flex-start;
  gap: 20px; /* 间距以防止重叠 */
}

.style-picker-container {
  width: 200px; /* 你可以根据需要调整 */
}

.todo-section {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
  padding: 20px;
}

.saved-list-section {
  margin-left: 20px;
  width: 300px; /* 你可以根据需要调整 */
}

.title-input {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 10px;
  width: 100%;
  text-align: center;
  border: none !important;
  border-bottom: 2px solid rgb(56, 97, 119) !important;
  outline: none;
}

.trash-btn,
.save-btn {
  background-color: #fff;
  border: none;
  margin-right: 10px;
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
