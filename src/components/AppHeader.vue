<template>
  <div class="app-header">
    <h1>Newslet</h1>
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
   $height: 32px;

   background: red;
   height: $height;

   h1 {
     float: left;
     line-height: $height;
     padding: 0 10px;
     font-weight: bold;
     color: white;
   }

   button {
     float: right;
   }
 }
</style>
