<template>
    <header></header>
    <section>
      <UserTable 
      endpoint="https://jsonplaceholder.typicode.com/users"
      v-bind:search="true"
      v-bind:sorting="true"
      v-bind:pagination="true"
      />
    </section>
    <footer></footer>
</template>

<script>
import UserTable from './components/UserTable.vue'

export default {
  name: 'App',
  components: {
    UserTable
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
