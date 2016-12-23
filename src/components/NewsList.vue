<template>
  <div class="news-list">
    <div>News list!</div>
    <li v-for="item in news">
      {{item}}
    </li>
  </div>
</template>

<script>
import NewsService from '../services/News'

export default {
  name: 'news-list',

  data: () => ({
    news: []
  }),

  created () {
    NewsService.getNewsStream().subscribe(news => {
      this.news = news;
    })
  }
}
</script>

<style scoped>

</style>
