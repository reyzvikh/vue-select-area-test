<template>
  <select-area-container>
    <div class="root"></div>
  </select-area-container>
</template>

<script>
  import SelectAreaContainer from './SelectAreaContainer';

  export default {
    name: 'Main',
    components: { SelectAreaContainer },
  };
</script>

<style scoped>
  .root {
    border: 1px solid black;
    background-color: #f1f1f1;
    height: 600px;
    width: 800px;
    background-image: url("../assets/sky.png");
  }
</style>
