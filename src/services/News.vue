<script>
import SourcesService from './Sources';
import Data from './Data';
import Rx from 'rxjs/Rx';

export default {
  STATUS: {
    NEW: 0,
    READ: 1,
    READING_LIST: 2,
    SKIPPED: 3
  },

  getNewsStream () {
    return SourcesService.getSourcesStream()
      .map(feeders => {
        return Rx.Observable.combineLatest(feeders.map(
          feeder => feeder.getNewsStream()
        )).map(feedersNews => {
          let news = [];
          feedersNews.forEach(moreNews => {
            news = news.concat(moreNews);
            news.sort((a, b) => b.timestamp - a.timestamp);
          });
          return news;
        });
      }).switch();
  },

  getStatusesStream () {
    return Data.getStream('statuses');
  },

  setStatus (entry, status) {
    Data.getStream('statuses') // TODO: const that
      .first()
      .subscribe((statuses = {}) => {
        statuses[entry.id] = status;
        Data.set('statuses', statuses);
      });
  }
}
</script>
