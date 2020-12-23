<script>
import Vue from 'vue';
import VueDeckgl from '@/vue-deckgl.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueDeckgl
  }
});
</script>

<template>
  <div id="app">
    <vue-deckgl />
  </div>
</template>
