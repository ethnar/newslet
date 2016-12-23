<template>
  <div class="news-list">
    <div v-for="item in news" class="entry">
      <div class="thumbnail">
        <img :src="item.imageUrl"/>
      </div>
      <a :href="item.href" class="intro">
        <div class="title">{{item.title}}</div>
        <pre class="content">{{item.content}}</pre>
        <span class="timestamp">{{item.timestamp}}</span>
      </a>
    </div>
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

<style scoped lang="scss">

  .entry {
    display: flex;
    height: 100px;
    border-bottom: 1px solid #ddd;

    .thumbnail {
      max-width: 30%;
      min-width: 30%;
      img {
        float: left;
        max-width: 100%;
        max-height: 100%;
      }
    }

    .intro {
      flex-grow: 1;
      display: flex;
      flex-direction: column;
      text-decoration: inherit;
      color: inherit;
      padding: 10px 5px;

      .title {
        text-align: left;
        font-weight: bold;;
      }

      .content {
        font-size: 85%;
        text-align: left;
        color: #999;
        padding-top: 5px;
        flex-grow: 1;
        overflow: hidden;
      }

      .timestamp {
        text-align: right;
        font-size: 70%;
        color: #ccc;
      }
    }
  }

</style>
