<template>
  <div>
    <label>Due date
      <input type="date" v-model="newTask.dueDate">
    </label>
    <label>Title
      <input type="text" v-model.trim="newTask.title" @keyup.enter="addTask">
      <input type="date" v-model="newTask.dueAt">
      <button type="submit" @click="saveTask(newTask)">SAVE</button>
      <button @click="logout">LOGOUT</button>
    </label>
    <select v-model="newTask.priority">
      <option value="high">high</option>
      <option value="medium">medium</option>
      <option value="low">low</option>
    </select>
    <button @click="addTask">Add Task</button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      newTask: {}
    }
  },

  created () {
    this.newTask = this.getNewTask()
  },

  methods: {
    addTask () {
      // this.taskList.push(this.newTask)
      this.$emit('taskAdded', this.newTask)
      this.newTask = this.getNewTask()
    },

    getNewTask () {
      const now = new Date()
      const today = now.toISOString().substr(0, now.toISOString().indexOf('T'))
      console.log(now)
      return {
        id: null,
        category: '',
        dueDate: today,
        isComplete: false,
        priority: 'medium',
        title: ''
      }
    },
    async saveTask (newTask) {
      const response = await axios.post(`${this.baseURL}/tasks`, newTask)
      return response.data.data
    },
    async updateTask (task) {
      const response = await axios.put(`${this.baseURL}/tasks/${task.id}`, task)
      return response.data.data
    },
    async deleteTask (task) {
      const response = await axios.delete(`${this.baseURL}/tasks/${task.id}`)
      return response.data.data
    }
  }
}
</script>

<style>

</style>
