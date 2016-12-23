<script>
import SourcesService from './Sources';
import Rx from 'rxjs/Rx';

export default {
  getNewsStream () {
    return SourcesService.getSourcesStream()
      .map(feeders => {
        return Rx.Observable.combineLatest(feeders.map(
          feeder => feeder.getNewsStream()
        )).map(feedersNews => {
          let news = [];
          feedersNews.forEach(moreNews => {
            news = news.concat(moreNews);
          });
          return news;
        });
      }).switch();
  }
}
</script>
