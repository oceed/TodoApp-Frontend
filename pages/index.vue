<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-100">
    <div class="bg-white p-6 rounded-lg shadow-md w-full max-w-2xl">
      <h1 class="text-2xl font-bold text-center mb-6">Todo List</h1>
      <div class="grid grid-cols-1 gap-2 md:grid-cols-3 mb-4">
        <input v-model="newTodo.title" placeholder="Title" class="border p-2 rounded-md md:col-span-2">
        <button @click="createTodo" :disabled="!newTodo.title" class="bg-blue-500 hover:bg-blue-700 text-white p-2 rounded-md md:col-span-1">Add</button>
      </div>
      <div class="flex justify-between">
        <div class="w-full mr-2">
          <h2 class="text-xl font-bold text-center mb-4">Pending Todos</h2>
          <table class="table-auto w-full">
            <thead>
              <tr>
                <th class="px-4 py-2">Title</th>
                <th class="px-4 py-2">Status</th>
                <th class="px-4 py-2">Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="todo in pendingTodos" :key="todo.id" class="border-t">
                <td class="border px-4 py-2">{{ todo.title }}</td>
                <td class="border px-4 py-2">{{ todo.status }}</td>
                <td class="border px-4 py-2 text-center">
                  <button @click="completeTodo(todo.id)" class="bg-green-500 hover:bg-green-700 text-white p-2 rounded-md">Complete</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="w-full ml-2">
          <h2 class="text-xl font-bold text-center mb-4">Completed Todos</h2>
          <table class="table-auto w-full">
            <thead>
              <tr>
                <th class="px-4 py-2">Title</th>
                <th class="px-4 py-2">Status</th>
                <th class="px-4 py-2">Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="todo in completedTodos" :key="todo.id" class="border-t">
                <td class="border px-4 py-2">{{ todo.title }}</td>
                <td class="border px-4 py-2">{{ todo.status }}</td>
                <td class="border px-4 py-2 text-center">
                  <button @click="deleteTodo(todo.id)" class="bg-red-500 hover:bg-red-700 text-white p-2 rounded-md">Delete</button>
                </td>
              </tr>
            </tbody>
          </table>  
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const todos = ref([])
const newTodo = ref({
  title: '',
  status: 'pending'
})

const pendingTodos = computed(() => todos.value.filter(todo => todo.status === 'pending'))
const completedTodos = computed(() => todos.value.filter(todo => todo.status === 'completed'))


async function fetchTodos() {
  try {
    const res = await axios.get('http://localhost:4000/todos')
    todos.value = res.data
  } catch (error) {
    console.error('Error fetching todos:', error)
  }
}

async function createTodo() {
  try {
    const res = await axios.post('http://localhost:4000/todos', newTodo.value)
    todos.value.push(res.data)
    newTodo.value.title = ''
    newTodo.value.status = ''
  } catch (error) {
    console.error('Error creating todo:', error)
  }
}

async function completeTodo(id) {
  try {
    const todo = todos.value.find(todo => todo.id === id)
    if (todo) {
      const res = await axios.put(`http://localhost:4000/todos/${id}`, { ...todo, status: 'completed' }, {
        headers: { 'Content-Type': 'application/json' }
      })
      const index = todos.value.findIndex(todo => todo.id === id)
      todos.value[index] = res.data
    }
  } catch (error) {
    console.error('Error completing todo:', error)
  }
}

async function deleteTodo(id) {
  try {
    await axios.delete(`http://localhost:4000/todos/${id}`)
    todos.value = todos.value.filter(todo => todo.id !== id)
  } catch (error) {
    console.error('Error deleting todo:', error)
  }
}

fetchTodos()
</script>
