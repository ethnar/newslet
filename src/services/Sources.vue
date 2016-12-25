<script>
import Feeder from '../model/Feeder';
import Data from './Data';

export default {
  getSourcesStream () {
    return Data
      .getStream('sources')
      .map(sources => {
        sources = sources || [];
        return sources.map(source => new Feeder(source));
      });
  },

  addSource (properties) {
    return this.getSourcesStream()
      .first()
      .toPromise()
      .then(feeders => {
        Data.set(
          'sources',
          feeders
            .map(feeder => feeder.toJSON())
            .concat([properties])
        );
      });
  },

  removeSource (sourceId) {
    console.log(sourceId);
  }
}
</script>
