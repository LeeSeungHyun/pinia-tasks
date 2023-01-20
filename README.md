# pinia-tasks

Vue 공식 문서에서 권고하고 있는 새로운 상태 관리 라이브러리

## Pinia vs Vuex

- Vuex와 달리 Pinia에는 mutation이 없다.
- Typescript 지원

## Store 생성 및 사용

TaskStore.js
```javscript
import { defineStore } from 'pinia'

export const useTaskStore = defineStore('taskStore', {
  state: () => ({
    tasks: [],
    loading: false
  }),
  getters: {
    favs() {
      return this.tasks.filter(t => t.isFav)
    },
    favCount() {
      return this.tasks.reduce((p, c) => {
        return c.isFav ? p + 1 : p
      }, 0)
    },
    totalCount: (state) => {
      return state.tasks.length
    }
  },
  actions: {
    async getTasks() {
      this.loading = true
      const res = await fetch('http://localhost:3000/tasks')
      const data = await res.json()

      this.tasks = data
      this.loading = false
    },
    async addTask(task) {
      this.tasks.push(task)

      const res = await fetch('http://localhost:3000/tasks', {
        method: 'POST',
        body: JSON.stringify(task),
        headers: {'Content-Type': 'application/json'}
      })

      if (res.error) {
        console.log(res.error)
      }
    },
    async deleteTask(id) {
      this.tasks = this.tasks.filter(t => {
        return t.id !== id
      })

      const res = await fetch('http://localhost:3000/tasks/' + id, {
        method: 'DELETE',
      })

      if (res.error) {
        console.log(res.error)
      }
    },
    async toggleFav(id) {
      const task = this.tasks.find(t => t.id === id)
      task.isFav = !task.isFav

      const res = await fetch('http://localhost:3000/tasks/' + id, {
        method: 'PATCH',
        body: JSON.stringify({ isFav: task.isFav }),
        headers: {'Content-Type': 'application/json'}
      })

      if (res.error) {
        console.log(res.error)
      }
    }
  }
})
```

TaskForm.vue
```javscript
<template>
  <form @submit.prevent="handleSubmit">
    <input 
      type="text" 
      placeholder="I need to..."
      v-model="newTask"
    >
    <button>Add</button>
  </form>
</template>

<script>
import { ref } from 'vue'
import { useTaskStore } from '../stores/TaskStore'
export default {
  setup() {
    // store 객체 접근
    const taskStore = useTaskStore()
    const newTask = ref('')
    const handleSubmit = () => {
      if (newTask.value.length > 0) {
        // Action 함수 실행
        taskStore.addTask({
          title: newTask.value,
          isFav: false,
          id: Math.floor(Math.random() * 1000000)
        })
        newTask.value = ""
      }
    }
    return { handleSubmit, newTask }
  }
}
</script>
```