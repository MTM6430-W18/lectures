<template>
  <section>
    <template v-if="tasks.length > 0">
      <TaskList
        listTitle="All Tasks"
        :tasks="tasks"
        @toggleDone="toggleDone"
        @removeTask="removeTask"
      />
    </template>
    <p v-else>There are no tasks ... add a new one!</p>
  </section>
</template>

<script>
// We will use this reusable component to minimize duplicating layout markup.
import TaskList from '@/components/TaskList'

export default {
  props: ['tasks'],
  components: { TaskList },
  methods: {
    // We will simply pass these events up to the parent App.vue component,
    // since that is where we are storing the master list.
    toggleDone (task) { this.$emit('toggleDone', task) },
    removeTask (task) { this.$emit('removeTask', task) }
  }
}
</script>

<style>

</style>
