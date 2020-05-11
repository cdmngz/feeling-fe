<template>
  <div class="container">
    <h3>Aqui estamos en Home.vue previo al component HelloWorld</h3>
    <HelloWorld/>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'Home',
  components: {
    HelloWorld
  }
}
</script>
