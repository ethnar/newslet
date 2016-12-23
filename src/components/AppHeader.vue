<template>
  <div class="app-header">
    <h1>HEADER!</h1>
    <button @click="addSource();">Add another</button>
  </div>
</template>

<script>
import SourcesService from '../services/Sources';

export default {
  name: 'app-header',

  methods: {
    addSource () {
      SourcesService.addSource({
        rssUrl: 'http://www.polygon.com/rss/index.xml',
        name: 'Polygon'
      });
    }
  }
}
</script>

<style scoped lang="scss">
 .app-header {
   background: red;
 }
</style>
