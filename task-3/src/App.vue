<template>
  <div id="app">
    <header class="app-header">
      <h1>Kanban Board</h1>
    </header>
    <Board />
  </div>
</template>


<script>
import Board from './components/Board.vue';


export default {
  name: 'App',
  components: {
    Board,
  },
};
</script>
