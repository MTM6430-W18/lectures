<template>
  <div id="app">
    <div v-if="isLoggedIn">
    <!-- We need some navigation links to switch between views -->
    <nav>
      <ul>
        <!--
          The RouterLink component comes with VueRouter.
          The 'to' prop is the URL route.
        -->
        <li><router-link to="/">All</router-link></li>
        <li><router-link to="/active">Active</router-link></li>
        <li><router-link to="/completed">Completed</router-link></li>
      </ul>
    </nav>
    <h1 class="title">Vue Todos</h1>
    <add-task-form @taskAdded="addTask" />
    <!--
      The RouterView is a placeholder that VueRouter uses to know
      where to insert the designated component for a given URL.
      In this case we are passing our taskList as a prop to each route's
      component and we are listening for user events.  This way we are still
      only mainting one master list in the App.vue component.
     -->
    <router-view
      :tasks="taskList"
      @toggleDone="toggleDone"
      @removeTask="removeTask"
    />
      </div>
    <login-page v-else @saveApiTokens="saveApiTokens" />
  </div>
</template>

<script>
import axios from 'axios'
import AddTaskForm from '@/components/AddTaskForm'
import LoginPage from '@/pages/LoginPage'

export default {
  name: 'App',
  components: { LoginPage, AddTaskForm },
  data () {
    return {
      baseURL: 'https://vue-todos.robertmckenney.ca/api',
      taskList: [
        // { id: 1234, title: 'Learn Vue', isComplete: true, priority: 'medium' },
        // { id: 1235, title: 'Learn Vue Router', isComplete: false, priority: 'medium' },
        // { id: 1236, title: 'Learn Vuex', isComplete: false, priority: 'medium' },
        // { id: 1237, title: 'Learn Vue DevTools', isComplete: true, priority: 'medium' }
      ]
    }
  },

  created () {
    axios
      .get(`${this.baseURL}/priorities`)
      .then(response => {
        this.priorityOptions = response.data.data.length
          ? response.data.data
          : []
      })
      .catch(error => { console.error(error) })
  }
  },

    methods: {
    refreshTasks () {
      axios
        .get('/todos', this.axiosOptions)
        .then(response => { this.taskList = response.data.data })
        .catch(error => { this.handleError(error) })
    },
    async getTasks () {
      try {
        const response = await axios.get(`${this.baseURL}/tasks`)
        this.taskList = response.data.data
      } .catch (error => {
          if (error.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            console.log(error.response.data);
            console.log(error.response.status);
            console.log(error.response.headers);
          } else if (error.request) {
            // The request was made but no response was received
            // `error.request` is an instance of XMLHttpRequest in the browser
            // and an instance of http.ClientRequest in node.js
            console.log(error.request);
          } else {
            // Something happened in setting up the request that triggered an Error
            console.log('Error', error.message);
          }
          console.log(error.config);
        })
      },
      async getTask (taskId) {
        const response = await axios.get(`${this.baseURL}/tasks/${taskId}`)
        return response.data.data
      },


    addTask (task) {
      axios
        .post('/todos', task, this.axiosOptions)
        .then(response => { this.taskList.push(response.data.data) })
        .catch(error => { this.handleError(error) })
    },
    toggleDone (task) {
      task.isComplete = !task.isComplete
      axios
        .put(`/todos/${task.id}`, task, this.axiosOptions)
        .catch(error => { this.handleError(error) })
    },
    removeTask (task) {
      axios
        .delete(`/todos/${task.id}`, this.axiosOptions)
        .then(response => {
          const taskIndex = this.taskList.findIndex(t => t.id === task.id)
          this.taskList.splice(taskIndex, 1)
        })
        .catch(error => { this.handleError(error) })
    },

    logout () {
      this.api.accessToken = null
      this.api.expiresAt = null
      localStorage.removeItem('todoApiTokens')
    },

    handleError (error) {
      if (error.response) {
        // The request was made and the server responded with a status code
        // that falls out of the range of 2xx
        switch (error.response.status) {
          case 400: {
            this.errorMessage = 'Both login name and password are required.'
            break
          }
          case 401: {
            this.errorMessage = 'Incorrect username or password'
            break
          }
          default:
            this.errorMessage = 'Sorry. There was an error processing your request.'
        }
      } else if (error.request) {
        // The request was made but no response was received
        console.log(error.request)
      } else {
        // Something happened in setting up the request that triggered an Error
        console.log('Error', error.message)
      }
      console.log(error.config)
    },
    loadInitialData () {
      const apiTokens = JSON.parse(localStorage.getItem('todoApiTokens'))
      if (apiTokens) {
        this.api.accessToken = apiTokens.access_token
        this.api.expiresAt = apiTokens.expires_at
        if (this.isLoggedIn) this.refreshTasks()
      }
    }
  }
  computed: {
    isLoggedIn () {
      return this.api.accessToken && moment(this.api.expiresAt).isAfter()
    },
    axiosOptions () {
      return {
        baseURL: this.baseURL,
        headers: { 'Authorization': `Bearer ${this.api.accessToken}` }
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  width: 100%;
  max-width: 550px;
  margin: 60px auto;
}
ul {
  list-style-type: none;
  margin: 0;
  padding: 0;
}
nav ul {
  display: flex;
  justify-content: flex-start;
}
nav ul li {
  margin-right: 1em;
}
</style>
