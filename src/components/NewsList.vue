<template>
  <div class="news-list">
    <div class="updates-available" :class="{visible: updateAvailable}" @click="updateList();">
      More news available
    </div>
    <div v-for="(item, index) in news" class="entry" :class="statusClass(item)" ref="items" :data-index="index">
      <div class="thumbnail">
        <img :src="item.imageUrl"/>
      </div>
      <a :href="item.href" class="intro" @click="startReading(item);">
        <div class="heading">
          <div class="title">{{item.title}}</div>
        </div>
        <pre class="content">{{item.content}}</pre>
        <span class="timestamp">{{item.timestamp}}</span>
      </a>
    </div>
    <div class="empty-space" :style="{opacity: allReadOpacity }">
      You're up-to-date on all news
    </div>
  </div>
</template>

<script>
import NewsService from '../services/News';

export default {
  name: 'news-list',

  data: () => ({
    news: [],
    statuses: {},
    updateAvailable: false,
    filter: NewsService.STATUS.NEW,
    allReadOpacity: 1
  }),

  created () {
    const newsStream = NewsService.getNewsStream();
    newsStream.subscribe(news => {
      this.latestNews = news;
      if (this.news.length === 0) {
        this.updateList();
      } else {
        this.updateAvailable = (news[0] && news[0].id !== this.mostRecentKnownNews.id);
      }
    });

    NewsService.getStatusesStream()
      .subscribe((statuses = {}) => {
        this.statuses = JSON.parse(JSON.stringify(statuses));
      });

    window.addEventListener('scroll', this.handleScroll);
  },

  destroyed () {
    window.removeEventListener('scroll', this.handleScroll);
  },

  methods: {
    startReading (entry) {
      NewsService.setStatus(entry, NewsService.STATUS.READ);
    },

    statusClass (entry) {
      const status = this.getStatus(entry);
      switch (status) {
        case NewsService.STATUS.READ: return 'status-read';
        case NewsService.STATUS.READING_LIST: return 'status-reading-list';
        case NewsService.STATUS.SKIPPED: return 'status-skipped';
        case NewsService.STATUS.NEW:
        default:
          return 'status-new';
      }
    },

    handleScroll () {
      if (this.$refs.items) {
        this.$refs.items.forEach(dom => {
          if (dom.offsetTop + dom.offsetHeight < window.scrollY + 5) {
            const index = dom.getAttribute('data-index');
            const status = this.getStatus(this.news[index]);
            if (status === NewsService.STATUS.NEW) {
              NewsService.setStatus(this.news[index], NewsService.STATUS.SKIPPED);
            }
          }
        });
      }
      this.updateAllReadOpacity();
    },

    updateAllReadOpacity () {
      let pixelsToBottom = (document.body.offsetHeight - (window.scrollY + window.innerHeight));
      this.allReadOpacity = Math.max(1 - pixelsToBottom / 50, 0);
    },

    updateList () {
      window.scrollTo(0, 0);
      this.mostRecentKnownNews = this.latestNews[0];
      this.news = this.latestNews.filter(entry => this.getStatus(entry) === this.filter);
      this.updateAvailable = false;
      setTimeout(() => {
        this.updateAllReadOpacity();
      });
    },

    getStatus (entry) {
      return this.statuses[entry.id] || NewsService.STATUS.NEW;
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
    z-index: 1;

    &.visible {
      top: 40px;
      opacity: 1;
      pointer-events: all;
    }
  }

  .empty-space {
    height: calc(100vh - 32px);
    font-weight: bold;
    display: table-cell;
    vertical-align: middle;
    text-align: center;
    width: 100vw;
  }

  .entry {
    display: flex;
    height: 100px;
    border-bottom: 1px solid #ddd;

    &.status-skipped {
      opacity: 0.4;
    }

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
        white-space: pre-wrap;
        line-height: 130%;
      }

      .timestamp {
        text-align: right;
        font-size: 70%;
        color: #ccc;
      }
    }
  }

</style>
