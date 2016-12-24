<template>
  <div class="news-list">
    <div class="updates-available" :class="{visible: updateAvailable}" @click="updateList();">
      More news available
    </div>
    <div v-for="item in news" class="entry" :class="statusClass(item.status)">
      <div class="thumbnail">
        <img :src="item.imageUrl"/>
      </div>
      <a :href="item.href" class="intro" @click="startReading(item);">
        <div class="heading">
          <div class="title">{{item.title}}</div>
          <div class="status-icon">{{item.status}}</div>
        </div>
        <pre class="content">{{item.content}}</pre>
        <span class="timestamp">{{item.timestamp}}</span>
      </a>
    </div>
  </div>
</template>

<script>
import NewsService from '../services/News';

export default {
  name: 'news-list',

  data: () => ({
    news: [],
    updateAvailable: false
  }),

  created () {
    const newsStream = NewsService.getNewsStream();

    newsStream.subscribe(news => {
      this.latestNews = news;
      if (this.news.length === 0) {
        this.news = news;
      } else {
        this.updateAvailable = (news[0] && news[0].id !== this.news[0].id);
      }
    });

    setTimeout(() => {
      this.updateAvailable = true;
    }, 1000);

    window.addEventListener('scroll', this.handleScroll);
  },

  destroyed () {
    window.removeEventListener('scroll', this.handleScroll);
  },

  methods: {
    startReading (entry) {
      NewsService.setStatus(entry, NewsService.STATUS.READ);
    },

    statusClass (status) {
      switch (status) {
        case NewsService.STATUS.NEW: return 'status-new';
        case NewsService.STATUS.READ: return 'status-read';
        case NewsService.STATUS.READING_LIST: return 'status-reading-list';
        case NewsService.STATUS.SKIPPED: return 'status-skipped';
      }
    },

    handleScroll () {
      console.log(window.scrollY);
    },

    updateList () {
      window.scrollTo(0, 0);
      this.news = this.latestNews;
      this.updateAvailable = false;
    }
  }
}
</script>

<style scoped lang="scss">
  .news-list {
    position: relative;
  }

  .updates-available {
    position: fixed;
    top: 0;
    padding: 9px;
    left: 50%;
    width: 250px;
    margin-left: -125px;
    box-shadow: 2px 2px 3px #444;
    color: white;
    background-color: #79d;
    cursor: pointer;
    opacity: 0;
    pointer-events: none;
    transition: all 0.4s linear;

    &.visible {
      top: 40px;
      opacity: 1;
      pointer-events: all;
    }
  }

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

      .heading {
        text-align: left;
        font-weight: bold;
        display: flex;

        .title {
          flex-grow: 1;
        }
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
