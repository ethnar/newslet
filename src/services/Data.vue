<script>
import Rx from 'rxjs/Rx';

let stream = null;

export default {
  getStream (property) {
    if (!stream) {
      stream = new Rx.ReplaySubject(1);
      stream.next(JSON.parse(window.localStorage.getItem('dataStorage')));
    }
    return stream
      .map(allData => allData ? allData[property] : undefined)
      .distinctUntilChanged(undefined, JSON.stringify);
  },

  set (property, value) {
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
