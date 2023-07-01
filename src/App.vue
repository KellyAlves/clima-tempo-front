<template>
  <q-img 
      :src="url"
      spinner-color="white"
      style="max-width: 300px"
    />
  <HelloWorld/>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import images from './assets/images/logo.png'
import { ref } from 'vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }, 
  setup() {
    const url = ref(images)
    return { url }
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
  margin-top: 30px;
}

</style>
