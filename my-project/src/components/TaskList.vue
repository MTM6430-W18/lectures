<template>
  <div>
    <h2>{{listTitle}}</h2>
    <div class="toolbar">
      <label for="priority-filter">Show priority</label>
      <select name="priority-filter" id="priority-filter" v-model="selectedPriority">
        <option value="">all</option>
        <option
          v-for="option in priorityOptions"
          :value="option"
          :key="option">{{option}}</option>
      </select>
    </div>
    <ul class="task-list">
      <li v-for="task in filteredTasks" :key="task.id" class="task-item">
        <span class="far"
          :class="{
            'fa-circle': ! task.isComplete,
            'fa-check-circle': task.isComplete
          }"
          @click="toggleDone(task)"></span>
        <span>{{ task.title }}</span>
        <span>{{ task.priority }}</span>
        <span class="far fa-trash-alt" @click="removeTask(task)"></span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  // We are not actually managing the task list in this component.
  // We are only displaying the list. The parent component (App.vue) is
  // passing down a reference to the list in the 'tasks' prop.
  props: ['tasks', 'listTitle'],
  data () {
    return {
      priorityOptions: ['low', 'medium', 'high'],
      selectedPriority: '',
      selectedCategory: ''
    }
  },
  computed: {
    filteredPriorityTasks () {
      // If the selectedFilter is 'all', the value is set to an empty string
      // which will evaluate to false. In that case, we will reuturn the
      // entire task list. Otherwise we use the array.filter() method to return
      // only the tasks with the selected priority.
      return (!this.selectedPriority)
        ? this.tasks
        : this.tasks.filter(task => task.priority === this.selectedPriority)
    },
    filteredCategoryTasks () {
      return (!this.selectedCategory)
        ? this.tasks
        : this.tasks.filter(task => task.category === this.selectedCategory)
    },
    filteredTasks () {
      return (!this.selectedPriority && !this.selectedCategory)
        ? this.tasks
        : this.tasks.filter(task => {
          let matched = false
          if (this.selectedPriority && this.selectedCategory) {
            if (task.priority === this.selectedPriority &&
                task.category === this.selectedCategory) {
              matched = true
            }
          } else if (this.selectedPriority) {
            if (task.priority === this.selectedPriority) matched = true
          } else if (this.selectedCategory) {
            if (task.category === this.selectedCategory) matched = true
          }
          return matched
        })
    }
  },
  methods: {
    // Since this component does not own the task list data, we need to
    // notify the parent component that the user wants to toggle the
    // completion status of the task.  We do that by emitting a custom event,
    // and passing up the affected task with the event. We will listen for
    // this event in the parent component and call the appropriate method to
    // make the change to the data.
    toggleDone (task) {
      this.$emit('toggleDone', task)
    },
    removeTask (task) {
      this.$emit('removeTask', task)
    }
  }
}
</script>

<style>
/* We will add styles here later. */
.task-item {
  display: grid;
  grid-template-columns: auto 1fr auto auto;
  grid-gap: 1rem;
  align-items: center;
}
.toolbar {
  margin-bottom: 1rem;
}
</style>
