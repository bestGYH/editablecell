<template>
  <div class="demo">
    <Editablecell/>
  </div>

</template>

<script>
import Editablecell from './view/editablecell.vue'

export default {
  name: 'app',
  components: {
    Editablecell
  }
}
</script>

<style>
  .demo {
    padding: 24px;
  }

</style>
