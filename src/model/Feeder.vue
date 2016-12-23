<script>
import Rx from 'rxjs/Rx';
// import RSS from 'feed-reader';

class Feeder {
  constructor (args) {
    this.name = args.name;
    this.rssUrl = args.rssUrl;
    this.stream = null;
  }

  toJSON () {
    return {
      name: this.name,
      rssUrl: this.rssUrl
    }
  }

  getNewsStream () {
    if (!this.stream) {
      this.stream = new Rx.ReplaySubject(1);

      window.fetch('http://cors-proxy.htmldriven.com/?url=' + this.rssUrl)
      .then(response => {
        return response.text()
      })
      .then(news => {
        this.stream.next(['news1', 'news2']);
      }).catch(e => window.alert('error retrieving news'));
    }
    return this.stream;
  }
}

Feeder.options = {}; // workaround?
export default Feeder;
</script>
