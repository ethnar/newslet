<script>
import Rx from 'rxjs/Rx';

let stream = null;

export default {
  getStream (property) {
    if (!stream) {
      stream = new Rx.ReplaySubject(1);
      stream.next(JSON.parse(window.localStorage.getItem('dataStorage')));
    }
    stream
      .map(allData => allData ? allData[property] : undefined)
      .distinctUntilChanged();
    return stream
      .map(allData => allData ? allData[property] : undefined)
      .distinctUntilChanged();
  },

  update (property, value) {
    let dataStorage;
    try {
      dataStorage = JSON.parse(
        window.localStorage.getItem('dataStorage')
      ) || {};
    } catch (e) {
      dataStorage = {};
    }
    dataStorage[property] = value;
    window.localStorage.setItem('dataStorage', JSON.stringify(dataStorage));
    stream.next(dataStorage);
  }
}
</script>
