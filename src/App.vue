<template>
  <div id="app">
    <h1>我的待办事项</h1>
    <ToDoList />
  </div>
</template>

<script>
import ToDoList from './components/ToDoList.vue';

export default {
  components: {
    ToDoList
  }
};
</script>

<style>
#app {
  text-align: center;
  font-family: Arial, sans-serif;
}
</style>
