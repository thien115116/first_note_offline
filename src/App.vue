<template>
  <div class="container-fluid">
    <div class="mb-5">
      <button class="btn btn-primary">
        Access Offline Mode
      </button>
    </div>
    <SimpleForm />
    </div>
</template>

<script>
import SimpleForm from './components/Form/SimpleForm.vue'

export default {
  name: 'App',
  components: {
    SimpleForm
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
