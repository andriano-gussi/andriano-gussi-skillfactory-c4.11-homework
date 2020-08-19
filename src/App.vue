<template>
  <div id="app">
    <TodosTable/>
  </div>
</template>

<script>
import TodosTable from './components/TodosTable.vue'

export default {
  name: 'App',
  components: {
    TodosTable
  }
}
</script>

